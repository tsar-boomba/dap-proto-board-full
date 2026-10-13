# TODO — Spin 2

Datasheet sources used in the full review (2026-10-11): ESP32-S3 Series Datasheet v2.2, ESP32-S3 Hardware Design Guidelines, nPM1300 PS v1.1, TPS63031 (SLVS696D), TPS22917, TAD5212 (SLASFC1A), ST USBLC6-2, Same Sky UJ20-C-H-G-SMT-1-P16-TR / MSD-11-A / SJ-352X-SMT, Abracon ABM8, Adafruit EYESPI guide.

## Schematic

### Power tree
- [x] U7 (TPS63031) EN / PS/SYNC / VINA tied to C37 only — matches TI Fig. 6; VINA is fed from VIN by an internal resistor (Fig. 4). Note: PS/SYNC tied high (via VINA) = power-save **disabled** (forced PWM) at all times.
- [x] L2 swapped to 1.5 µH Murata DFE201610E-1R5M=P2 (`LCSC PN` C668311), footprint `Inductor_SMD:L_Murata_DFE201610P`.
- [x] Power-sheet title updated.

### I2C
- [x] nPM1300 bus R10/R11 and TAD5212 bus R12/R13 are 10k.

### SD card (U1 MSD-11-A, U6 TPS22917)
- [x] C35 (1 nF) CT → VIN is correct per the TPS22917 datasheet. SD_ON pull-down R16 added.
- [x] **Move SD pull-ups R5–R9 from +3V3 to SD_VCC.** (done) When U6 is off its QOD pulls SD_VCC low (150 Ω, TPS22917 §9.5), so the five 10k pull-ups to +3V3 back-feed the card through its I/O clamp diodes (~1.3 mA wasted, current injected into the card). In USB-OTG download mode the ROM also drives MTDO high and MTCK/GPIO38 low (ESP32-S3 HW guidelines Table 12) into the unpowered card. Firmware should also tri-state SD lines while SD is off.
- [ ] (Optional) MSD-11-A footprint has a `WP` ("side spring") pad with no symbol pin, so it's unconnected. Datasheet only lists it as a mechanical spring; tie to GND if you want it grounded.

### ESP32-S3 (FH4R2)
- [x] VDD_SPI untied from +3V3; C21 1 µF + C22 100 nF (matches HW guidelines §1.3.2).
- [x] **Add a 10k pull-up on GPIO0.** (R15) HW guidelines §1.3.9: "It is recommended to place a pull-up resistor at the GPIO0 pin." (internal weak pull-up exists, so low risk).
- [ ] (Optional) 499 Ω series on U0TXD (§1.3.7) and an LC filter on VDD3P3 (§1.3.2). Both are for RF harmonics; low priority since RF isn't used.

### Display power / strapping (2026-10-11)
- [x] (Fixed: CS → GPIO15, GPIO45 back on J5-11) **CRITICAL: `/disp_CS` is now on GPIO45, a strapping pin.** GPIO45 = 1 at reset selects VDD_SPI = 1.8 V (ESP32-S3 datasheet Table 3-4); its internal pull-down is only weak (Table 3-1). Adafruit EYESPI displays pull DISP_CS up to VIN (= +3V3) with 10k (1.69"/1.9" Eagle sources: R1 TFTCS→VIN). So with a display plugged in, the in-package 3.3 V flash/PSRAM gets 1.8 V and the chip won't boot. Move CS to a non-strapping GPIO (e.g. a J5 pin such as GPIO18).
- [x] (Done: U8, ON = `DISP_ON` on GPIO16, R17 100k pull-down, C40 1 nF CT→VIN, QOD shorted to VOUT) **Switch display power with a second TPS22917**, the same circuit as U6: VIN = +3V3, VOUT → J1-1, CT 1 nF to VIN, input/output caps. Drive ON from **GPIO45**: a load-switch ON input only loads the pin, so the strap stays 0, and the smart ON pull-down (~750 kΩ) keeps the display (and backlight) off through boot. Firmware: drive all display lines low or hi-Z while the rail is off, or they back-feed the display through its 10k pull-ups and input clamps (same as SD).

- [x] (Done: C41/C42) Put a 1 µF on +3V3 right at U8's VIN, and at U6's (TPS22917 §11: "CIN of 1 μF is sufficient"). +3V3 has plenty of bulk, but it may not be close to U8 or U6 on the PCB. Optional 100 nF–1 µF on DISP_VCC at J1; Adafruit displays have their own input cap.

### Debug / UART (found by fresh-context review)
- [x] **Break out UART0 TX.** (done) GPIO43 (U0TXD) is used as display CS, so the ROM boot log ends up on the FPC and toggles display CS on every boot (harmless while SCLK idles). Move display CS to a free GPIO and put GPIO43 on J5 (or a test pad) next to GPIO44.
- [x] (Superseded by U8) **1 kΩ pull-down on `/disp_LITE` (GPIO21)** so the backlight stays off until firmware drives it. The default 100 mA VBUS limit (nPM1300 §6.1.1) leaves little headroom with no battery. Adafruit EYESPI displays (1.69", 1.9", 2.0" Eagle sources) pull LITE up to 3.3 V with 10k into a BSS138 gate (V_GS(th) 0.8 V min, −2 mV/°C). A 1k pull-down gives 0.30 V, solidly off; 2.2k gives 0.60 V, which is marginal at temperature. Driving it high costs 3.3 mA from GPIO21.

### Status LEDs (D1/D2)
- [x] **Set LED part numbers (Vf ≤ ~2.8 V).** Anodes are on +3V3 and nPM1300 LED pins need ≥ 0.5 V (PS v1.1 Table 25). Both of these are JLC **Basic** parts in 0805, and their max Vf is from the datasheet:
  - Red: **NCD0805R1** (C84256), Vf 1.5 / 2.0 / 2.6 V min/typ/max @ 20 mA.
  - Yellow: **KT-0805Y** (C2296), Vf 1.8–2.4 V @ 10 mA.
  - Change the footprint from `LED_1206_…_HandSolder` to `LED_SMD:LED_0805_2012Metric` (or `_HandSolder`). The 1206 equivalents (KT-1206R C49018, KT-1206Y C2298) are Extended parts.
  - Avoid C72043 (the common JLC green): it's InGaN, Vf 3.3 V. Blue and white won't work either.

### Display (J1, EYESPI)
- [x] J1 pin order matches Adafruit EYESPI pinout (1 VCC … 18 GPIO2).
- [x] **Display RESET (J1-8) → GPIO37.** Free on the FH4R2: GPIO33–37 are only reserved for octal flash/PSRAM (datasheet §2.3.4). Was: Fine only if your display module has its own power-on reset (Adafruit: "often there is an automatic-reset chip on the display"). Otherwise route it to a spare GPIO.

### TAD5212
- [x] Pinout, I2S direction, caps (AVDD/DREG/IOVDD 10 µF+0.1 µF, VREF 1 µF, MICBIAS 1 µF), ADDR→GND (I2C 0x50), and OUT1M+OUT2M→sleeve all match the datasheet (Fig. 6-16 / Fig. 8-1).
- [ ] (Optional) GPI1 (pin 12) is an unconnected digital input; datasheet doesn't state a default pull. Tie to GND to be safe.

### Connectors
- [x] USB-C (J2, UJ20-C-H-G-SMT-1-P16-TR): drawing says 0.8 mm PCB, but this exact part is proven on 1.6 mm boards. OK.
- [ ] **Battery (J4):** pin 1 = +BATT. JST PH packs vary by vendor; check your pack with a meter before plugging in (no reverse-polarity protection on board).

### BOM / housekeeping
- [x] MPNs added: U4 ESP32-S3FH4R2, U5 nPM1300-QEAA-R, U7 TPS63031DSKR.
- [ ] Project footprint libraries (`MSD-11-A`, `SFV18R_1STE1HLF`, TAD5212 `footprints`) point to `~/Downloads/...` in `fp-lib-table`. Copy them into the repo and use `${KIPRJMOD}` paths.
- [ ] README pinout still describes the RP2350 board.

### JLC sourcing (checked against JLC's parts API, 2026-10-11)
- [x] **Y1 → Epson TSX-3225 40.0000MF15Y-K3** (C17/C18 now 15 pF) (Mouser, 200 in stock). A reseller listing for MF15Y-K gives CL 10 pF, ±10 ppm tolerance and ±15 ppm stability; confirm on the Mouser page. Epson brief: ESR ≤ 40 Ω; pins #1/#3 are the crystal and #2/#4 the lid, which goes to GND, so it fits the current footprint.
  - **Change C17/C18 from 18 pF to 15 pF (Basic C1644).** Stray is ~3 pF (12 = 18/2 + 3), so C = 2 × (10 − 3) = 14 → 15 pF.
  - Set Y1's Value/Part Number to the new MPN.
  - ±15 ppm over temperature is slightly looser than Espressif's ±10 ppm guidance. That guidance is for RF; USB full-speed only needs ±2500 ppm, and RF is never enabled on this board.
- [x] **J2 `LCSC PN` C2988369 (removed) is GT-USB-7010ASV, not UJ20-C-H-G-SMT-1-P16-TR** (the UJ20 is C45082766). Make the value, LCSC number and footprint agree on whichever connector you've actually used.
- [x] Passives: chosen in JLC's web BOM tool, not in the schematic (user's choice). When picking there:
  - For the VSYS caps C27/C29/C31/C36 (and ideally all 10 µF), choose a **25 V** 10 µF (C96446). Nordic's BOM specifies 25 V; the 10 V C19702 loses a lot of capacitance at 5 V.
  - Keep C17/C18 C0G.
- [x] LEDs: see Status LEDs above (C84256 / C2296, both Basic).
- [x] **SW1–SW3:** chose C&K PTS810SJM250SMTR LFS, hand-placed (same land as the Würth footprint). the Würth 434133025816 (C5504987) has 3 in stock. No 4.2×3.2 mm switch is Basic or Preferred; JLC's only Basic switches are TS-1088 (4×3 mm) and TS-1187A (5.1 mm). **Drop-in Extended swap: Alps SKRP series**, same 4.2×3.2 mm J-lead body. Alps' recommended land is 3.1/5.2 mm × 1.5/2.8 mm, i.e. pads at ±2.075, ±1.075, 1.05×0.65 mm. That is exactly the KiCad Würth footprint, and pins 1–2 and 3–4 are paired the same way, so no footprint change is needed.
  - SKRPABE010 (C115360): 1.57 N, the same feel as the current Würth (1.6 N); ~45k stock.
  - SKRPACE010 (C139797): 2.55 N, for a firmer click; ~126k stock.
- [x] (Reverted to USBLC6) **U2 SRV05-4 is NOT a drop-in for the current wiring. Fix before ordering.** The schematic passes D± *through* U2: J2 D− → pin 1, pin 6 → /USB_D− → R3, and likewise D+ via pins 3/4. That only works on the USBLC6, whose pins 1↔6 and 3↔4 are joined internally (ST Fig. 1). The SRV05-4 has four independent channels (datasheet p.1), so USB would be open circuit. Fix: join `Net-(J2-D-)` to `/USB_D-` and `Net-(J2-D+)` to `/USB_D+`, so pins 1+6 are both on D− and 3+4 both on D+. On the PCB, route each line across both of its pads. Or revert to USBLC6-2SC6 (Extended).
- [ ] **J1 / U3 / other Extended parts are hand-placed from Mouser**, so JLC stock doesn't matter. Just confirm Mouser stock for SFV18R-1STE1HLF and TAD5212IRGER when ordering. Old JLC notes: the Amphenol SFV18R-1STE1HLF (C3170088) has 0 stock. Stocked 18P 0.5 mm bottom-contact flip-top options: FC-05A18P11H20 (C23398842), BX-FPC05-2H18PX (C18077648), FPC0.5B-WTG-18P (C22360340). Each needs its own footprint, and its contact side must match the EYESPI cable.
  - U3: TAD5212IRGER (C45068920) has 1 in stock. Pre-order or use JLC global sourcing, or consider the TAD5212QRGERQ1 (C47078053, 64 stock; automotive grade, pin compatibility not verified).
- [x] U6: TPS22917DBVR (C2681320, ~23k stock) instead of the DBVT (C2680344, 480 stock). Same part, bigger reel.
- [x] Remaining Extended parts (hand-placed):
  - ESP32-S3FH4R2 C3013940
  - nPM1300-QEAA-R C7466043
  - TPS63031DSKR C15516
  - L2 C668311
  - MSD-11-A C22467016
  - SJ-3523-SMT-TR C4991872
  - JST S2B-PH-SM4-TB C295747
- [x] J4 battery (hand-placed, bought from Mouser): **genuine JST S2B-PH-SM4-TB(LF)(SN)**, Mouser 306-S2BPHSM4TBLFSN (~52k in stock). This is the exact part the KiCad `JST_PH_S2B-PH-SM4-TB` footprint is drawn for, so no change is needed; just set the J4 Value/Part Number.
- [ ] J5 (1×14 THT socket): hand-solder or leave unpopulated (debug only).

## Firmware / bring-up

### Charging & VBUS (nPM1300) — already handled; keep in sync with hardware
- Decided: **no thermistor.** NTC (pin 18) stays tied to GND (Nordic's documented no-thermistor setup).
- On every boot/reset: disable the NTC check (`BCHGDISABLESET`), set `BCHGVTERM` and the charge current (`BCHGISET*`), then `BCHGENABLESET`. Charging is disabled by default after any reset.
- On every VBUS-detected event: write `VBUSINILIM0`, then `TASK.UPDATE.ILIMSW`. The limit reverts to 100 mA on replug or reset. CC1/CC2 status (`USBCDETECTSTATUS`) tells you whether 1.5 A / 3 A is available.

### Spin-1 bring-up (root cause of "no USB" still open)
- [ ] **Check Y1 on the spin-1 board.** If it was assembled from the BOM, it's the 12 MHz C20625731 (see JLC sourcing). With 12 MHz instead of 40 MHz, the USB PHY clock is wrong, so USB would never enumerate, and the ROM UART log would come out at ~34560 baud instead of 115200. Probe XTAL_P/N with a scope or read the crystal's marking. If it's 12 MHz, swap in a 40 MHz / 12 pF crystal.
- [ ] **Force download mode for the first flash:** hold SW1 (GPIO0), tap SW2 (reset), release SW1. Espressif HW guidelines §1.5: "If the flash is empty, set the chip … to Joint Download Boot mode."
- [ ] If still nothing: read the UART0 ROM boot log. **TX (GPIO43) is not on J5**; it goes only to the display FPC as `/disp_CS` (J1-9). On spin 1, probe the J1 pin-9 pad with no display fitted (115200 baud, 3.3 V). At ~34560 baud the log points to a 12 MHz crystal.
- [ ] Measure VSYS (~5 V on USB) and +3V3 on the spin-1 board.
- [ ] If the chip seems dead/unstable: suspect ESP32 exposed-pad soldering (see PCB section). The EP is the chip's only GND.

### Other
- **Never initialize Wi-Fi/BT.** LNA_IN is unconnected (no antenna); Espressif §1.3.6: leaving RF floating is OK only if the RF stack isn't started, "operation without an antenna may result in unstable behavior or potential damage."
- GPIO42/MTMS (`SD_ON`): drive high to power the SD card (R16 holds it off during boot).
- GPIO45 (J5-11) is a strapping pin: keep it floating or low at boot. If it's high, VDD_SPI switches to 1.8 V and the 3.3 V in-package flash/PSRAM won't boot.
- TAD5212 output mode: set `OUT1x_CFG = 101` (pseudo-differential, OUT1M = VCOM, OUT2M = VCOM sense) to match the jack wiring. Note the jack sleeve carries VCOM, not GND.

## PCB (do all at once, after schematic items are done)

- [x] ESP32-S3 EP land is 4×4 mm (KiCad `QFN-56-1EP_7x7mm_P0.4mm_EP4x4mm`), smaller than the FH4R2's 5.7 mm EPAD. **This is intentional**, to leave room to break out GPIOs. No overlap with the signal pads (3.05 mm vs 2.85–2.90 mm). On spin 1, inspect EP solder if the chip misbehaves.
- [x] **Update PCB from schematic** (F8). Pulls in: R3/R14 USB series resistors, R16, I2C pull-up values, VDD_SPI untie + C21/C22, L2 footprint change, plus any new schematic items above.
- [x] **Re-place/reroute around U7.** L2 grows from 0603 to 2.0×1.6 mm; keep L2, C36 (VIN), C37 (VINA) and C38/C39 (VOUT) tight to the IC per TI §11.
- [ ] **Verify the L2 footprint** (`L_Murata_DFE201610P` is for the DFE201610**P**) against the DFE201610**E** land pattern. (Murata datasheet not obtained in review.)
- [ ] USBLC6 (U2): if the VBUS trace from U2 to C25 is long, add 100 nF VBUS→GND right at U2 (ST datasheet §3).
- [ ] TAD5212: route the OUT2M sense trace to the jack sleeve pin itself (Fig. 6-16 "common mode sensed through OUT2M").
- [x] (2026-10-11: 0 errors, 0 unconnected; only silkscreen warnings plus a J5 lib-footprint-mismatch warning) **Re-fill zones and run DRC** (incl. schematic parity) — expect 0 unconnected, 0 shorts.
- [ ] **Regenerate `production/` outputs** (gerbers, drill, BOM, CPL, `netlist.ipc`). The committed `netlist.ipc` / `bom.csv` are stale.
