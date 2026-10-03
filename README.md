# Pattern Memory Game

A memory game for an **8x8 LED matrix**, an **analog joystick** and an **Arduino**, built for a robotics workshop.

The matrix shows a random pattern for a few seconds. The pattern disappears, and you have to **recreate it from memory** by moving a blinking cursor with the joystick and pressing the joystick button to lock each square. The further you go, the smaller the squares get, so the game keeps getting harder.

---

## Table of contents

1. [What you need](#1-what-you-need)
2. [How the game works](#2-how-the-game-works)
3. [Connections (wiring)](#3-connections-wiring)
4. [Installing the Arduino IDE](#4-installing-the-arduino-ide)
5. [Installing the LedControl library](#5-installing-the-ledcontrol-library)
6. [Uploading the game](#6-uploading-the-game)
7. [Testing your hardware](#7-testing-your-hardware)
8. [Playing the game](#8-playing-the-game)
9. [Customizing the game](#9-customizing-the-game)
10. [Troubleshooting](#10-troubleshooting)
11. [Workshop tips and ideas](#11-workshop-tips-and-ideas)

---

## 1. What you need

| Part | Notes |
|---|---|
| Arduino Uno or Nano | Any ATmega328P board works |
| USB cable | The correct type for your board (Uno: USB-B, Nano: Mini-USB or Micro-USB) |
| 8x8 LED matrix with **MAX7219** driver | The module with 5 pins: VCC, GND, DIN, CS, CLK |
| **HW-504** analog joystick module | 5 pins: GND, +5V, VRx, VRy, SW |
| Breadboard | For sharing 5V and GND |
| Jumper wires | Male-to-male and/or male-to-female depending on your modules |
| A computer | Windows, macOS or Linux |

> **Photo of the finished setup:** add your own photo to an `images/` folder and it will show up here.
>
> `![Finished setup](images/setup.jpg)`

---

## 2. How the game works

1. A random pattern is shown on the matrix for a few seconds.
2. The matrix goes blank. A **blinking cursor** appears in the middle.
3. Move the cursor with the **joystick**.
4. **Press the joystick button** to lock a square (it stays lit). Press again on a locked square to unlock it.
5. When you have locked **as many squares as the pattern had**, the game checks your answer automatically.
6. **Correct:** the matrix shows your **new level number**, then the next round starts.
   **Wrong:** a cross appears, the correct pattern is shown, and the game restarts at level 1.

### Difficulty stages

The size of one "square" shrinks as you level up. Each new stage also starts with fewer squares, so the jump in difficulty is fair.

| Stage | Levels | Square size | Grid | Squares to remember |
|---|---|---|---|---|
| 1 | 1 - 4 | 2x2 LEDs (big squares) | 4 x 4 = 16 | 3 to 6 |
| 2 | 5 - 8 | 2x1 LEDs (rectangles) | 4 x 8 = 32 | 3 to 6 |
| 3 | 9 - 16 | 1x1 LED (single pixel) | 8 x 8 = 64 | 4 to 11 |

What the three stages look like (`#` = lit LED):

```
Stage 1 (2x2)       Stage 2 (2x1)       Stage 3 (1x1)
##..##..            ##..##..            #...#...
##..##..            ......##            ....#...
..##....            ##......            ..#.....
..##....            ..##....            .....#..
```

---

## 3. Connections (wiring)

![Wiring diagram](wiring_diagram.svg)

**Do all wiring with the Arduino unplugged from USB.**

### MAX7219 LED matrix to Arduino

| Matrix pin | Arduino pin |
|---|---|
| VCC | 5V |
| GND | GND |
| DIN | D12 |
| CS | D10 |
| CLK | D11 |

### HW-504 joystick to Arduino

| Joystick pin | Arduino pin |
|---|---|
| GND | GND |
| +5V | 5V |
| VRx | A0 |
| VRy | A1 |
| SW | D2 |

### Wiring tips

- Both modules need 5V and GND. The Arduino has only a few of these pins, so connect 5V and GND to the **power rails of the breadboard** and branch from there.
- Many matrix modules have two 5-pin headers. Connect to the **input** side (labelled **DIN**), not the output side (**DOUT**).
- Leave pin **A5 unconnected**. The code reads it to get random numbers so that every game is different.
- The joystick button (SW) uses the Arduino's built-in pull-up resistor, so you do not need an extra resistor.

> **Photo of the wiring:** `![Wiring photo](images/wiring.jpg)`

---

## 4. Installing the Arduino IDE

The Arduino IDE is the free program used to write code and upload it to the board.

### Step 1: Download

1. Open **https://www.arduino.cc/en/software** in your browser.
2. Under **Arduino IDE**, choose the download for your system:
   - **Windows:** "Windows Win 10 and newer, 64 bits" (installer)
   - **macOS:** the Apple Silicon or Intel version that matches your Mac
   - **Linux:** the AppImage or ZIP file
3. You may see a donation page. You can choose **"Just download"**.

### Step 2: Install

- **Windows:** double-click the downloaded `.exe` and follow the installer. Accept the driver prompts if they appear.
- **macOS:** open the `.dmg` and drag **Arduino IDE** into the **Applications** folder.
- **Linux:** make the AppImage executable (`chmod +x <file>.AppImage`) and run it, or extract the ZIP and run the program inside.

### Step 3: Connect the Arduino

1. Plug the Arduino into the computer with the USB cable.
2. Open the Arduino IDE.
3. Go to **Tools > Board** and choose your board:
   - **Arduino Uno:** *Arduino AVR Boards > Arduino Uno*
   - **Arduino Nano:** *Arduino AVR Boards > Arduino Nano*
4. Go to **Tools > Port** and choose the port that appeared when you plugged in the board:
   - Windows: `COM3`, `COM4`, etc.
   - macOS: `/dev/cu.usbmodem...` or `/dev/cu.usbserial...`
   - Linux: `/dev/ttyUSB0` or `/dev/ttyACM0`

### Extra steps for Nano clones or cheap boards

- If the port does not appear, the board probably uses a **CH340** USB chip. Install the CH340 driver (search for "CH340 driver" for your operating system), then re-plug the board.
- For an **Arduino Nano**, if uploading fails, go to **Tools > Processor** and select **ATmega328P (Old Bootloader)**.
- On Linux, if the port is not accessible, add your user to the `dialout` group (`sudo usermod -a -G dialout $USER`) and log in again.

---

## 5. Installing the LedControl library

The game uses the **LedControl** library to talk to the MAX7219 matrix.

1. In the Arduino IDE, click the **Library Manager** icon on the left side (the books icon), or go to **Sketch > Include Library > Manage Libraries...**
2. In the search box, type **LedControl**.
3. Find **LedControl** by **Eberhard Fahle** in the results.
4. Click **Install**. If it asks about dependencies, click **Install all**.
5. Wait until it shows **INSTALLED**.

> Make sure you pick the one by **Eberhard Fahle**. Other libraries have similar names and will not work with this code.

---

## 6. Uploading the game

1. Keep these files together in a folder named **`pattern_memory_game`**:
   - `pattern_memory_game.ino`
   - `README.md`
   - `wiring_diagram.svg`

   > The Arduino IDE needs the `.ino` file to be inside a folder with the **same name**. This is already set up correctly in this download.
2. Open the Arduino IDE and go to **File > Open...**, then pick `pattern_memory_game.ino`.
3. Check that the correct **Board** and **Port** are selected (**Tools** menu).
4. Click the **Verify** button (check mark) to compile. Wait for "Done compiling".
5. Click the **Upload** button (right arrow). Wait for **"Done uploading"**.
6. The matrix flashes fully on **twice**, and then the first pattern appears.

---

## 7. Testing your hardware

If something does not work, test each part on its own. Create a new sketch (**File > New Sketch**), paste one of these, and upload it.

### Test A: LED matrix

A single LED should move across the matrix one by one.

```cpp
#include <LedControl.h>
LedControl lc = LedControl(12, 11, 10, 1);  // DIN, CLK, CS

void setup() {
  lc.shutdown(0, false);
  lc.setIntensity(0, 5);
  lc.clearDisplay(0);
}

void loop() {
  for (int r = 0; r < 8; r++) {
    for (int c = 0; c < 8; c++) {
      lc.clearDisplay(0);
      lc.setLed(0, r, c, true);
      delay(100);
    }
  }
}
```

If **all LEDs stay on** the whole time, the matrix is not receiving data. See the troubleshooting section.

### Test B: Joystick

Open **Tools > Serial Monitor** and set the speed to **9600 baud**.

```cpp
void setup() {
  Serial.begin(9600);
  pinMode(2, INPUT_PULLUP);
}

void loop() {
  Serial.print("X: "); Serial.print(analogRead(A0));
  Serial.print("  Y: "); Serial.print(analogRead(A1));
  Serial.print("  Button: "); Serial.println(digitalRead(2));
  delay(200);
}
```

What you should see:

| Action | X | Y | Button |
|---|---|---|---|
| Joystick centered | about 500 | about 500 | 1 |
| Pushed to one end | near 0 or 1023 | near 0 or 1023 | 1 |
| Button pressed | - | - | 0 |

---

## 8. Playing the game

1. Power the Arduino over USB (or any 5V supply).
2. Watch the pattern carefully while it is on screen.
3. When the matrix goes blank, move the blinking cursor with the joystick.
4. Press the joystick down to lock a square. Press again on a locked square to remove it.
5. Lock exactly as many squares as were in the pattern. The game checks automatically.
6. Get it right and you will see your new level number. How far can you get?

The cursor **wraps around** the edges, so moving off the right edge brings you to the left.

---

## 9. Customizing the game

All settings are at the top of `pattern_memory_game.ino`.

| Setting | What it does | Default |
|---|---|---|
| `MAX_LEVEL` | Highest level of the game | 16 |
| `STAGE1_LEVELS` | Number of levels with 2x2 squares | 4 |
| `STAGE2_LEVELS` | Number of levels with 2x1 rectangles | 4 |
| `START_CELLS_S1`, `S2`, `S3` | Number of squares in the first level of each stage | 3, 3, 4 |
| `SHOW_TIME_BASE` | Base time (ms) the pattern is shown | 2000 |
| `SHOW_TIME_PER_CELL` | Extra time (ms) added per square | 300 |
| `LEVEL_DISPLAY_TIME` | How long (ms) the level number is shown | 1500 |
| `MOVE_DELAY_S1`, `S2`, `S3` | Cursor speed in each stage (lower = faster) | 220, 190, 150 |
| `INVERT_X`, `INVERT_Y` | Flip joystick directions | false |
| `MIRROR_COLUMNS` | Mirror the matrix image (and the digits) | false |
| `lc.setIntensity(0, 5)` in `setup()` | LED brightness, 0 to 15 | 5 |

---

## 10. Troubleshooting

| Problem | What to try |
|---|---|
| **All LEDs are on and nothing changes** | The matrix is not receiving data, or the sketch did not upload. Check that you are using the **DIN** side, that DIN/CLK/CS are on D12/D11/D10, and that GND is connected. Re-seat the jumper wires (cheap wires are often faulty). Run Test A. |
| **Compile error: `LedControl.h: No such file`** | The library is not installed. Follow section 5. |
| **Upload fails or the port is missing** | Check the USB cable (some cables are charge-only). Select the right board and port. Install the CH340 driver for clones. For a Nano, try *ATmega328P (Old Bootloader)*. |
| **Cursor moves in the wrong direction** | Set `INVERT_X` and/or `INVERT_Y` to `true`. |
| **Picture or digits look mirrored** | Set `MIRROR_COLUMNS` to `true`. |
| **Picture is rotated 90 degrees** | Physically rotate the matrix module. Then adjust `INVERT_X`/`INVERT_Y` if the joystick directions feel wrong. |
| **Cursor drifts by itself** | Run Test B. If the centered values are far from 500 (for example below 300 or above 700), the joystick is faulty or badly wired. |
| **Button press does nothing** | Check the SW wire on D2. In Test B the button should read 0 when pressed. |
| **Same pattern every time you restart** | Make sure pin A5 is left unconnected. |
| **Matrix is very bright or dim** | Change the number in `lc.setIntensity(0, 5);` (0 to 15). |

---

## 11. Workshop tips and ideas

- Let participants wire everything themselves, then run Test A and Test B before uploading the game.
- Have a colour-coded wire for each signal (the wiring diagram uses red for 5V, black for GND).
- Challenge participants to change the difficulty: more squares per stage, shorter display time, or a different stage order.
- Extension ideas:
  - Add a buzzer for correct and wrong answers.
  - Add a second matrix for a two-player mode.
  - Keep and show a high score.
  - Add a start screen that waits for a button press.

---

## Credits

- LED matrix control: the [LedControl](https://github.com/wayoda/LedControl) library by Eberhard Fahle.
- Built with the Arduino platform.

Have fun and happy building!
