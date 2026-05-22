# Bomb DAP Full Proto Board

The culmination of all of my learning during the Spring '26 semester. All the way from making a simple dev board for the TAD5212 DAC, to making a RP2350 board, and now this. Everything combined on one 6-layer board.

# V1 Pinouts

- GPIO0 -> PSRAM_CS
- GPIO1 -> npm1300 GPIO0
- GPIO2 -> DISP_IO_ON (level-shifter enable)
- GPIO3 -> Display Backlight
- GPIO4 -> Display SCLK (PIO SPI)
- GPIO5 -> Display MOSI (PIO SPI)
- GPIO6 -> Display MISO (PIO SPI)
- GPIO7 -> Display DC & SDA
- GPIO8 -> Display CS & SCL
- GPIO(9-11) -> Broken out
- GPIO12 -> SD MISO (SPI1)
- GPIO13 -> SD CS
- GPIO14 -> SD SCLK (SPI1)
- GPIO15 -> SD MOSI (SPI1)
- GPIO16 -> npm1300 SDA (I2C0)
- GPIO17 -> npm1300 SCL (I2C0)
- GPIO18 -> SD ON (load switch & level-shifter enable)
- GPIO19 -> TAD GPIO1
- GPIO20 -> TAD SDA (PIO I2C)
- GPIO21 -> TAD SCL (PIO I2C)
- GPIO(22-26) -> Broken out
- GPIO27 -> TAD DIN (PIO I2S)
- GPIO28 -> TAD FSYNC (PIO I2S)
- GPIO29 -> TAD BCLK (PIO I2S)
