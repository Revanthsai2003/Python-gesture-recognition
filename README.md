# Python-gesture-recognition
using python


import cv2 
import mediapipe as mp
import pyautogui
import time 

def count_fingers(lst):
    cnt = 0

    thresh = (lst.landmark[0].y * 100 - lst.landmark[9].y * 100) / 2

    if (lst.landmark[5].y * 100 - lst.landmark[8].y * 100) > thresh:
        cnt += 1

    if (lst.landmark[9].y * 100 - lst.landmark[12].y * 100) > thresh:
        cnt += 1

    if (lst.landmark[13].y * 100 - lst.landmark[16].y * 100) > thresh:
        cnt += 1

    if (lst.landmark[17].y * 100 - lst.landmark[20].y * 100) > thresh:
        cnt += 1

    if (lst.landmark[5].x * 100 - lst.landmark[4].x * 100) > 6:
        cnt += 1

    return cnt 

# Open the webcam for capturing video
cap = cv2.VideoCapture(0)

# Initialize drawing utilities from mediapipe
drawing = mp.solutions.drawing_utils

# Initialize mediapipe Hands object
hands = mp.solutions.hands
hand_obj = hands.Hands(max_num_hands=1)

# Initialize variables for gesture r ecognition
start_init = False 
prev = -1

# Main loop for continuously processing video frames
while True:
    # Record the current time for timing purposes
    end_time = time.time()
    
    # Read a frame from the webcam and flip it horizontally
    _, frm = cap.read()
    frm = cv2.flip(frm, 1)

    # Process the frame to detect hand landmarks using mediapipe
    res = hand_obj.process(cv2.cvtColor(frm, cv2.COLOR_BGR2RGB))

    # Check if hand landmarks are detected
    if res.multi_hand_landmarks:

        # Get landmarks for the first detected hand
        hand_keyPoints = res.multi_hand_landmarks[0]

        # Count the number of fingers visible in the hand gesture
        cnt = count_fingers(hand_keyPoints)

        # Check if the finger count has changed since the last frame
        if not(prev == cnt):
            # If this is the first frame with a new finger count, record the start time
            if not(start_init):
                start_time = time.time()
                start_init = True

            # If enough time has passed since the last keyboard action, perform a new action
            elif (end_time - start_time) > 0.2:
                if cnt == 1:
                    pyautogui.press("right")
                
                elif cnt == 2:
                    pyautogui.press("left")

                elif cnt == 3:
                    pyautogui.press("up")

                elif cnt == 4:
                    pyautogui.press("down")

                elif cnt == 5:
                    pyautogui.press("space")

                # Update the previous finger count and reset the start flag
                prev = cnt
                start_init = False

        # Draw hand landmarks on the frame
        drawing.draw_landmarks(frm, hand_keyPoints, hands.HAND_CONNECTIONS)

    # Display the processed frame with hand landmarks
    cv2.imshow("window", frm)

    # Check for the 'Esc' key press to exit the program
    if cv2.waitKey(1) == 27:
        cv2.destroyAllWindows()
        cap.release()
        break
