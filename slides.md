---
# You can also start simply with 'default'
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: ./images/hwl.webp
# some information about your slides (markdown enabled)
title: Reverse Engineering Denuvo in Hogwarts Legacy
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
# seoMeta:
#  ogImage: https://cover.sli.dev
---

# Reverse Engineering<br>![denuvo-logo](./images/denuvo-logo.png) in ![hwl-logo](./images/hwl-logo.png)

---

## Who am I?

<div grid="~ cols-2 gap-2" m="t-2">

- Maurice Heumann
- Cybersecurity Engineer @ Thales
- Twitter: @momo5502

<img  src="./images/me.png" />
</div>

---

## Agenda

- Understanding Denuvo
- The Cracking Strategy
- Technical Analysis
- Performance Analysis


---

## What is Denuvo?

<div grid="~ cols-2 gap-2" m="t-2">

- Anti tamper solution by Irdeto
- Protects existing DRM/licensing solutions
  - e.g. Steam, Origin, ...

```mermaid
flowchart BT
    game(🎮 Game Executable)
    drm("🔒 DRM Layer (Steam)")
    denuvo(🛡️ Denuvo Anti-Tamper)
    drm --> game
    denuvo --> drm
```

</div>

---

## How does it work?

```mermaid

sequenceDiagram
    participant Player
    participant Denuvo as Denuvo Server
    participant Steam as Steam Server
    
    Player->>Denuvo: Hardware Fingerprint + Steam Ticket
    Denuvo->>Steam: Validate Steam Ticket
    Steam->>Denuvo: Ticket OK
    Denuvo->>Player: Generate Denuvo Token for Hardware Fingerprint
    Player->>Player: Run Game with Ticket
```


---

1. Deuvo generates hardware fingerprint
2. Denuvo/Steam generates a ticket -> proof of game ownership
3. Game sends steam ticket + fingerprint to server
4. Server validates steam ticket
5. Server generates a denuvo token and sends it back
6. Game runs and uses denuvo token to decrypt values at runtime

---

## Why is Denuvo so strong?

- Protection is unique per game
  - Different fingerprints, different patterns, ... 
- Usually games are packed when protected
- Denuvo doesn't pack
- Instead code is lifted and virtualized in a VM (not reversed)
-  sequences are inserted that validate the fingerprint (likely decryption)
- this causes tight coupling of game and denuvo code --> hard to separate
- thousands of validations require thousands of hooks

---


# Let's crack the game!

**Two Possible Approaches**
1. Patch and reverse all decryptions --> insane amount of work
2. Method 2: Fingerprint simulation --> chosen approach (empress also chose that one)

**Why Fingerprint Simulation?**
- Goal: mimic different PC with a valid token
- Find all fingerprints
- Patch all fingerprints to mimic other PC
- Run game with token from other PC

![rounded](./images/bad-token.png)

---

## How to find fingerprint features?

Many possible ways

- Debugger
- Hypervisor
- Emulator --> ✅

---

## Why emulator?

Denuvo must communicate with OS, hardware, filesystem, ...
The game needs to grab information from somewhere.
Pretty much three ways:

- API calls
- Special instructions (CPUID, Syscall, ...)
- Memory

An emulator can instrument all that:

- It can trace all API calls
- It can trace all instructions
- It can hook all memory access

What the emulator can not: emulate graphics -> it won't be able to fully boot into the game
luckily:
Denuvo has two phases:

1. collection phase, before it talks to the server
2. runtime, when the game runs, after server communication

-> emulation analysis only needs to run until the server communication.

## Fingerprint features

Vary for each protected game
HWL had 7 Major Categories

We'll explore how I found them, how I patched them

--> actual number may vary

---

## 1. API calls

- GetVolumeInformationW
- GetUserNameW
- GetComputerNameW
- CryptGetProvParam
- CryptAcquireContextA
- CryptAcquireContextW
- CryptEnumProvidersW
- ExpandEnvironmentStringsA &rarr; %COMPUTERNAME%

--> Just hook them and return deterministic values

---

## 2. PEB

- OSMajorVersion
- OSMinorVersion
- NumberOfProcessors
- ImageSubsystemMajorVersion
- ImageSubsystemMinorVersion

--> Just unprotect and overwrite the data
  - could have undesired consequences, overwriting the os version or number of cores

---

## 3. Environment Peeks

PEB->ProcessParameters->Environment
essentially random peeks into the env vars

- 0x74
- 0x123
- 0x1d8
- 0x291

--> just unprotect and overwrite

---
transition: slide-up
---

## 4. CPUID

- 1
- 0x80000002
- 0x80000003
- 0x80000004

- load hypervisor -> custom CPUID vmexit handler for hogwarts legacy
- hides other features -> xgetbv -> patch leaf
- other features conditionally active that i might not have needed to patch

---
transition: slide-down
---

### What is a hypervisor?

---

## 5. KUSER_SHARED_DATA

- NtProductType
- ActiveProcessorCount
- SuiteMask
- ProductTypeIsValid
- NtMajorVersion
- NtMinorVersion
- NtBuildNumber
- ProcessorFeatures
- NumberOfPhysicalPages

---

--> hard to patch
  - find all places -> ideally HWBP + exception handler
  - non-linear stack -> wrote a debugger that attaches to the game and traces using HWBP
  - no guarantee i'll ever have all locations

  - dynamic hook creation
  - redirect memory load to fake memory region
  - disassemble all load instruction
  - analyze and replicate memory source (scale-index-base)
  - replicate instruction (xor, add, mov, ...)


---
transition: slide-up
---

## 6. Inline syscalls

NtQuerySystemInformation &rarr; SystemBasicInformation

ntdll exports are parsed to find syscall ID

--> Inline syscalls
  - KUSD approach doesn't work -> mini integrity checks on instructions
    - instruction bytes are read and computed into other calculations
    - bytes need to stay intact
      --> hypervisor + ept hooks -> redirect syscalls to custom handler that replays original data
      --> syscall hooks would've also worked, but my hypervisor couldn't do that at the time

---
transition: slide-down
---

## Hypervisor -> shadow hooking

---

# The last one... FML
3 months...

---

### 7. Import integrity

- --> Advapi32.dll
- addresses of these values in IAT
- changing them invalidates the token, so aslr changes on a reboot might invalidate it.

* CryptAcquireContextA
* CryptGetProvParam
* GetUserNameW
* GetVolumeInformationW

--> insanely hard to find. why?
-> regular memory access, nothing special
-> game reads import table all the time, nothing suspicious
-> usually import is used for execution, not in denuvo case

--> simple to patch
- trampolinee at fixed VA that redirects to the original value
- requires that the VA is available, which it should be

---

## What does that leave us with?

--> game runs, but semi stable - why?
 -> sampling KUSD may miss values
 -> patching CPUID can destabilize system
 -> overwriting PEB can also destabilize 

--> 2k hooks. can we do something with that? 
-> we can analyze when the hooks are triggered to see in which situations the game executes denuvo code --> performance reasoning

---

## Performance Reasoning

- For me, impossible to make detailed measurements --> I would need game without denuvo and with denuvo
- denuvo changes a lot, each game is protected differently, even different versions of the game, each integration is different.
- denuvo has a dedicated team that performs integration into games
- prior analysis mostly meaningless, has to be looked at for each game invidivually

* each of my 2000 hooks prints when it's called
* if no print occurs, no denuvo verification code runs --> very likely no possibility of performance impacts
* video a few prints during normal gameplay
* lots of prints during transition/loadscreen

---

## Performance?

<Youtube id="6JriEmiZ1t0" width="720" height="405" />
