
```
# Virtual Hand Gesture Keyboard

This Python application uses OpenCV, MediaPipe, and pynput to create a virtual keyboard that can be controlled using hand gestures. The application captures video from your webcam, detects hand landmarks, and allows you to press virtual keys by moving your index finger and middle finger.

## Features
- Detects hand landmarks using MediaPipe.
- Displays a virtual keyboard on the screen.
- Allows pressing of keys by using hand gestures.
- Visual feedback when a key is pressed.

## Prerequisites
- Python 3.x
- Install the required libraries using pip:
  ```bash
  pip install opencv-python mediapipe pynput numpy
  ```

## How to Use

1. **Clone the repository**:
   ```
   git clone https://github.com/yourusername/virtual-hand-gesture-keyboard.git
   cd virtual-hand-gesture-keyboard
   ```

2. **Run the application**:
   ```
   python app.py
   ```

3. **Control the virtual keyboard**:
   - Use your webcam to capture your hand gestures.
   - The application will detect your hand and display a virtual keyboard.
   - Move your index and middle fingers to press keys.

## Code Overview

Here's a brief overview of the code in `app.py`:

```
import cv2
import mediapipe as mp
from pynput.keyboard import Controller
from time import sleep
import math
import numpy as np

# Initialize webcam, MediaPipe Hands, and drawing utilities
camIndex = 0
cap = cv2.VideoCapture(camIndex)
mpHands = mp.solutions.hands
Hands = mpHands.Hands()
mpDraw = mp.solutions.drawing_utils

# Define the keyboard layout
keys = [["Q", "W", "E", "R", "T", "Y", "U", "I", "O", "P"],
        ["A", "S", "D", "F", "G", "H", "J", "K", "L", ";"],
        ["Z", "X", "C", "V", "B", "N", "M", ",", ".", "/"]]
keyboard = Controller()

class Store():
    def __init__(self, pos, size, text):
        self.pos = pos
        self.size = size
        self.text = text
    
def draw(img, storedVar):
    for button in storedVar:
        x, y = button.pos
        w, h = button.size
        cv2.rectangle(img, button.pos, (x + w, y + h), (64, 64, 64), cv2.FILLED)  # greyish color
        cv2.putText(img, button.text, (x + 10, y + 43), cv2.FONT_HERSHEY_PLAIN, 3, (255, 255, 255), 2)
    return img

StoredVar = []
for i in range(len(keys)):
    for j, key in enumerate(keys[i]):
        StoredVar.append(Store([60 * j + 10, 60 * i + 10], [50, 50], key))

while cap.isOpened():
    success, img = cap.read()
    if not success:
        break

    cvtImg = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    results = Hands.process(cvtImg)
    lmList = []

    if results.multi_hand_landmarks:
        for img_in_frame in results.multi_hand_landmarks:
            mpDraw.draw_landmarks(img, img_in_frame, mpHands.HAND_CONNECTIONS)
        for id, lm in enumerate(results.multi_hand_landmarks[0].landmark):
            h, w, c = img.shape
            cx, cy = int(lm.x * w), int(lm.y * h)
            lmList.append([cx, cy])

    if lmList:
        for button in StoredVar:
            x, y = button.pos
            w, h = button.size
 
            if x < lmList[8][0] < x + w and y < lmList[8][1] < y + h:
                cv2.rectangle(img, (x - 5, y - 5), (x + w + 5, y + h + 5), (0, 0, 255), cv2.FILLED)
                x1, y1 = lmList[8][0], lmList[8][1]
                x2, y2 = lmList[12][0], lmList[12][1]  # landmarks 8 and 12 for index and middle finger tips
                l = math.hypot(x2 - x1 - 30, y2 - y1 - 30)
                print(l)
                # Check if fingers are close enough to indicate a "click"
                if not l > 63:
                    keyboard.press(button.text)
                    cv2.rectangle(img, (x - 5, y - 5), (x + w + 5, y + h + 5), (0, 255, 0), cv2.FILLED)
                    sleep(0.15)

    img = draw(img, StoredVar)
    cv2.imshow("Virtual Keyboard", img)

    if cv2.waitKey(1) == 113:  # Press 'Q' to quit
        break

cap.release()
cv2.destroyAllWindows()
```

## Adjustments

- **Key Press Threshold**:
  - The value `l > 63` determines the sensitivity of detecting a "click". Adjust this value if necessary.
  
- **Drawing and Text Position**:
  - Ensure the text position and rectangle sizes are appropriate for different screen resolutions.

## Contributions
Contributions are welcome! Please fork the repository and submit a pull request with your changes.

## Acknowledgements
- [OpenCV](https://opencv.org/)
- [MediaPipe](https://mediapipe.dev/)
- [pynput](https://pypi.org/project/pynput/)

