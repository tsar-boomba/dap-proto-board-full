# Bomb DAP Full Proto Board

The culmination of all of my learning during the Spring '26 semester. All the way from making a simple dev board for the TAD5212 DAC, to making a RP2350 board, and now this. Everything combined on one 6-layer board.

# V1 Pinouts

- GPIO0 -> PSRAM_CS
- GPIO1 -> npm1300 GPIO0
- GPIO2 -> DISP_IO_ON (level-shifter enable)
- GPIO3 -> Display Backlight
- GPIO4 -> Display SCLK
- GPIO5 -> Display MOSI
- GPIO6 -> Display MISO
- GPIO7 -> Display DC & SDA
- GPIO8 -> Display CS & SCL
- GPIO(9-11) -> Broken out
- GPIO12 -> SD MISO
- GPIO13 -> SD CS
- GPIO14 -> SD SCLK
- GPIO15 -> SD MOSI
- GPIO16 -> npm1300 SDA
- GPIO17 -> npm1300 SCL
- GPIO18 -> SD ON (load switch & level-shifter enable)
- GPIO19 -> TAD GPIO1
- GPIO20 -> TAD SDA
- GPIO21 -> TAD SCL
- GPIO(22-26) -> Broken out
- GPIO27 -> TAD DIN
- GPIO28 -> TAD FSYNC
- GPIO29 -> TAD BCLK
