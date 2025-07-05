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
transition: fade-out
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
# seoMeta:
#  ogImage: https://cover.sli.dev
---

<style scoped>
.slidev-layout {
    padding: 0px;
}

h1 {
  text-shadow: 0px 0px 0.3em rgba(0, 0, 0, 0.381);
  box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
}

h1 img.denuvo-logo {
  display: inline;
  transform: translateY(-0.05em);
  width: 180px;
  filter: invert() drop-shadow(0px 0px 0.1em rgba(0, 0, 0, 0.575));
}

h1 img.hwl-logo {
  display: inline;
  transform: translateY(-0.2em);
  width: 220px;
  filter: invert() drop-shadow(0px 0px 0.1em rgba(0, 0, 0, 0.575));
}
</style>
<h1 class="mt--70 backdrop-blur-xl p-9 text-shadow-3xl">
Reverse Engineering
<br>
<img class="denuvo-logo" src="./images/denuvo-logo.png" /> in <img class="hwl-logo" src="./images/hwl-logo.png" />
</h1>

---

# Who am I?

<div class="flex">

- Maurice Heumann
- Cybersecurity Engineer @ Thales
- Cracked & modded Call of Duty games (BOIII, XLabs)
- Twitter: @momo5502

<div class="flex-1 text-center">
<img class="rounded-2xl w-70 m-auto" src="./images/me.png" />
</div>
</div>

---

# Agenda

- Understanding Denuvo
- The Cracking Strategy
- Technical Analysis
- Performance Analysis

---

# What is Denuvo?

<div class="flex" m="t-2">

- **Anti-tamper solution** by Irdeto
- **Not a DRM itself** - protects existing DRM systems
  - Steam, Origin, Epic Games Store, etc.
- **Used by major publishers**: EA, Ubisoft, Square Enix, Capcom
- **Controversial**: Loved by publishers, hated by pirates
- **Business model**: License per game + server costs

<div class="text-center flex-1">

```mermaid
flowchart BT
    game(🎮 Game Executable)
    drm("🔒 DRM Layer (Steam)")
    denuvo(🛡️ Denuvo Anti-Tamper)
    drm --> game
    denuvo --> drm
```

</div>
</div>

---
clicks: 13
---

# How does it work?

<v-clicks every="0.5">

1. Hardware fingerprint is generated → Computername + Username + ...
2. Steam ticket generation → Proof of game ownership
3. Fingerprint & Steam ticket is sent to Denuvo server
4. Server validates steam ticket → Do you really own the game?
5. Server generates Denuvo token for the fingerprint
6. Game runs with Denuvo token

</v-clicks>

<div class="flex mt-6"
  v-motion
  :initial="{ x: 0, y: 300 }"
  :click-1="{ y: 0 }"
>
<div class="border-3 border-green p-4 rounded-lg">
🎮 Game
<div class="border-3 border-yellow rounded-md p-1 m-2"
  v-motion
  :initial="{ x: 0, y: 300 }"
  :click-3="{ y: 0 }"
  :click-7="{ x: 661 }"
  >
  🔍 Fingerprint
</div>

<div class="border-3 border-yellow rounded-md p-1 m-2"
  v-motion
  :initial="{ x: 0, y: 300 }"
  :click-5="{ y: 0 }"
  :click-7="{ x: 661 }"
>
  🎟️ Steam Ticket
</div>
<div
  class="absolute"
  v-motion
  :initial="{ x: 675, y: 300 }"
  :click-9="{ x: 675, y: -40 }"
>✅</div>
<div class="border-3 border-yellow rounded-md p-1 m-2 opacity-0"
>
  🔑 Denuvo Token
</div>
</div>

<div class="flex-1">
</div>

<div class="border-3 border-blue p-4 rounded-lg">
🌐 Denuvo Server

<div class="border-3 border-yellow rounded-md p-1 m-2 opacity-0"
  >
  🔍 Fingerprint
</div>

<div class="border-3 border-yellow rounded-md p-1 m-2 opacity-0">
  🎟️ Steam Ticket
</div>

<div class="border-3 border-white rounded-md p-1 m-2"
  v-motion
  :initial="{ x: 0, y: 300 }"
  :click-11="{ y: 0 }"
  :click-13="{ x: -661 }"
>
  🔑 Denuvo Token
</div>
</div>
</div>

---

# What is the fingerprint?

---

# What is the Denuvo token?

---

# What makes Denuvo so strong?

**Custom Protection Per Game:**

- Each game is unique
- Different fingerprints, patterns, validation
- No generic crack possible

**Advanced Code Protection:**

- No traditional packing - code remains accessible
- **Code virtualization** - critical sections run in custom VM
- **Tight integration** - Denuvo is mixed into game logic
- **Thousands of checks** - validations everywhere

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

# How to find fingerprint features?

Many possible ways

- Debugger
- Hypervisor
- Emulator --> ✅

---

# Why emulator?

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

# Fingerprint features

Vary for each protected game
HWL had 7 Major Categories

We'll explore how I found them, how I patched them

--> actual number may vary

---

# 1. API calls

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

# 2. PEB

- OSMajorVersion
- OSMinorVersion
- NumberOfProcessors
- ImageSubsystemMajorVersion
- ImageSubsystemMinorVersion

--> Just unprotect and overwrite the data

- could have undesired consequences, overwriting the os version or number of cores

---

# 3. Environment Peeks

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

# 4. CPUID

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

# What is a hypervisor?

---

# 5. KUSER_SHARED_DATA

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

# 6. Inline syscalls

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

# Hypervisor -> shadow hooking

---

# The last one...

Took me 3 months to find...

---

# 7. Import integrity

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

# It's running...

<img src="./images/running.png" />

---

# What does that leave us with?

--> game runs, but semi stable - why?
-> sampling KUSD may miss values
-> patching CPUID can destabilize system
-> overwriting PEB can also destabilize

--> 2k hooks. can we do something with that?
-> we can analyze when the hooks are triggered to see in which situations the game executes denuvo code --> performance reasoning

---

# Performance Reasoning

- For me, impossible to make detailed measurements --> I would need game without denuvo and with denuvo
- denuvo changes a lot, each game is protected differently, even different versions of the game, each integration is different.
- denuvo has a dedicated team that performs integration into games
- prior analysis mostly meaningless, has to be looked at for each game invidivually

* each of my 2000 hooks prints when it's called
* if no print occurs, no denuvo verification code runs --> very likely no possibility of performance impacts
* video a few prints during normal gameplay
* lots of prints during transition/loadscreen

---

# Performance Reasoning 2

<Youtube id="6JriEmiZ1t0" width="720" height="405" />
