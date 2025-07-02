# Reverse Engineering Denuvo Presentation Structure

## 1. Opening & Introduction (3-4 slides)

- **Title Slide**: Reverse Engineering Denuvo in Hogwarts Legacy
- **Speaker Introduction**: Who am I? (Maurice, Cybersecurity Engineer)
- **Agenda**: Clear roadmap of what we'll cover
- **Why This Topic Matters**: Brief context on game protection and reverse engineering

## 2. Understanding Denuvo (5-6 slides)

- **What is Denuvo?**
  - Anti-tamper solution by Irdeto
  - Protection layer architecture diagram
- **How Denuvo Works**
  - Authentication flow sequence diagram
  - Hardware fingerprinting concept
  - Server communication process
- **Why Denuvo is So Strong**
  - Code obfuscation and virtualization
  - Dynamic protection mechanisms
  - Server-side validation dependency
  - Unique per-game implementation
  - Anti-debugging and anti-analysis measures
- **The Challenge**
  - Why traditional cracking methods don't work well
  - Bad token example/screenshot

## 3. The Cracking Strategy (3-4 slides)

- **Two Possible Approaches**
  - Method 1: Patch and reverse all decryptions
  - Method 2: Fingerprint simulation (chosen approach)
- **Why Fingerprint Simulation?**
  - Strategy explanation
  - Goal: mimic different PC with valid token
- **Analysis Requirements**
  - What we need to find and understand

## 4. Technical Analysis (5-6 slides)

- **Choosing the Right Tool: Emulation**
  - Why emulator over debugger/hypervisor
  - What emulators can instrument
  - Denuvo's two-phase operation
- **Fingerprint Categories Discovery**
  - Overview of 7 major categories found in HWL
- **The Fingerprint Hunt: 7 Categories Discovered**
  - Interactive reveal approach (one category per click/animation)
  - Each category gets a "threat level" and brief explanation
  - Group by difficulty: Easy → Medium → Hard to patch
- **Category Spotlight** (1-2 detailed examples):
  - Pick 2-3 most interesting categories for deep dive
  - Show actual code/data examples
  - Explain the "aha!" moment of discovery

## 5. Implementation Challenges (4-5 slides)

- **Patching Strategy Overview**
  - Different approaches for different categories
- **Simple Patches**
  - API hooks
  - PEB/Environment data overwriting
  - Import integrity trampolines
- **Complex Patches**
  - KUSD dynamic hooking approach
  - CPUID hypervisor solution
  - Inline syscall EPT hooks
- **Technical Deep Dive**
  - Memory instruction analysis
  - Integrity check workarounds

## 6. Performance Analysis (2-3 slides)

- **The Performance Question**
  - Why measurement is difficult
  - Game-specific nature of Denuvo
- **Methodology**
  - Hook monitoring approach
  - What the data shows
- **Results Demonstration**
  - Video evidence
  - Gameplay vs loading screen impact

## 7. Conclusion & Takeaways (2-3 slides)

- **Key Findings**
  - Technical achievements
  - Performance insights
- **Broader Implications**
  - Anti-tamper evolution
  - Research methodology value
- **Q&A**

---

## Recommended Slide Count: 20-25 slides

## Estimated Duration: 25-35 minutes

### Presentation Tips:

- **Flow**: Each section builds on the previous
- **Balance**: Mix technical depth with accessibility
- **Engagement**: Use diagrams, code snippets, and video strategically
- **Pacing**: Allow time for complex technical concepts to sink in
- **Interaction**: Plan for questions throughout, not just at the end
