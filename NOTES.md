# Discovery Notes

Bench arc for the curious — how the fix was found and why the mechanism is what it is.

## The problem

A PC Engine Duo with a dead CD drive and TurboNanza installed. Pins 3 and 40 of the CD ASIC were lifted to kill the bus fight with the TED PRO. TED 2.5 functional but requiring a manual reset button press on every power-up. Several HuCards, including Jaseiken Necromancer, refused to boot entirely. Yōkai Dōchūki, for some reason, booted more or less consistently.

The shape of the problem pointed at the ASIC: something on the lifted pins was still poisoning the boot sequence despite being severed from the original bus.

## Vector map


### v0.5 — FAILED: A20 bridge through diode

Initial theory: route HuCard slot pin 24 (A20) through a 1N4148 back to ASIC pin 3 to emulate A20 arrival at the ASIC. Anode on cart pin 24, cathode on ASIC pin 3.

Result: solid screen, no boot.

Tied GND to pin 40 in hopes of completing the boot sequence or forcing it. same results

### v0.6 — ACCIDENTAL: floating diode


After v0.5 failed, undoing the install had two options:

- (a) Desolder the diode off ASIC pin 3 — easy
- (b) Desolder the wire off HuCard slot pin 24 — much easier

Option (b) was chosen. The diode stayed physically mounted on ASIC pin 3, with its anode leg now dangling (the cart-side wire removed).

Powered on to test if I had returned to where I started — it booted consistently, even Necromancer. Then stopped booting after a few power cycles.

This was the accident that cracked it open. My theory now was that with the anode floating, the diode's junction capacitance (~4pF) charged on the first XRESET edge and delivered a one-shot pulse to pin 3. That pulse depleted over subsequent cycles because there was nothing to refresh the charge.

### v0.7 — Insight

If a transient charge was resolving whatever was wrong at pin 3, a permanent reference should hold the state indefinitely. Ground the anode. Test.

### v1.0 — CONFIRMED

Cathode → ASIC pin 3, anode → GND. Clean auto-boot every cycle. Full HuCard compatibility. TED auto-launches. Necromancer boots clean on S-Video, looking like a million bucks on the Duo.

## Mechanism

With the diode installed as **cathode to pin 3, anode to GND**, the diode is a **one-way undershoot clamp**:

- **Reverse-biased when pin 3 ≥ 0V** → blocks current, pin 3 free to swing positive in its normal operating window
- **Forward-biased only when pin 3 tries to go below ~-0.7V** → snubs negative excursions back toward ground
 

### What the diode does (or at least what I think)

1. **Preserves positive-swing behavior on pin 3.** Something the ASIC (or residual coupling to the lifted pin stub, or the shared bus net) does on the positive side during boot matters. Hard ground crushes that and kills the fix.

2. **Clamps negative transients during XRESET release.** On the rail-stabilize window, the lifted ASIC pin's internal drivers and parasitic inductance of the pin stub can produce undershoot below GND. A sub-ground glitch on pin 3 couples back into the D91317GD's internal A20 input and into shared bus neighbors. The diode forward-conducts just in time to snub that glitch before it propagates.

3. **Protects the shared bus from pin 3 activity.** The one-way clamp lets pin 3 stay in its normal positive operating window but prevents any negative excursion from being injected into the shared cart-side net where a TED Pro (or similar Everdrive product) lives. Having pin 40 of the ASIC lifted and connected to GND frees up pin 1 of the cart slot to drive the right sound channel from the TED PRO freely.


## A note to anyone still reading

Hey, thanks for reading. My pending list is probably going to be delayed for quite a bit — I do own a Duo but I dont own a TED PRO and dont plan to buy one, as this fix was intended for a client who asked me to recap and install a TurboNanza on theirs.

Sadly the capacitor juice damaged the CD part of the console to the point that fixing it wasn't economically reasonable, so he decided to get a TED PRO instead and asked me to disable the CD for it. I thought there were going to be resources online about this but I found none, so I decided to tackle the problem myself. There is no ETA on when he is getting the TED PRO, and the Duo is already assembled and leaving my bench soon. So dont hold your breath for followups.

I appreciate any contribution though — feel free to contact me.

## Credit

This discovery builds on prior ASIC-silencing work in the PC Engine modding community — particularly the broader insight that the D91317GD must be silenced for TED Pro (and similar Everdrive products) to function cleanly on a bus with a live CD ASIC. 

For installs with a working CD drive, see ZAXOUR's **Duo Disguiser** — an approach that switches between CD and TED modes reversibly.
