# Sign Language Recognition

A browser-based **Sign Language Recognition** web application that uses a webcam to track hand landmarks with **MediaPipe Hands** and recognize a practical set of predefined signs using normalized hand geometry, hand position, and short temporal motion patterns.

## Live Demo

**GitHub Pages:** https://lohithl27.github.io/Sign-language/

The application is designed to run directly in a modern browser without a backend server. Camera processing, gesture classification, transcript generation, and speech synthesis happen locally in the browser.

> **Note:** This project is a practical heuristic gesture-recognition prototype. It is not a complete natural-language sign-language translator and should not be treated as a production accessibility or medical communication system.

## Features

- 📷 Live webcam-based hand tracking
- ✋ MediaPipe Hands 21-landmark tracking
- 👐 Support for up to two detected hands
- 🧠 Normalized finger geometry instead of simple screen-coordinate rules
- ⏱️ Short temporal motion history for movement-based signs
- 🎯 Stable-sign confirmation before adding a result to the transcript
- 🔊 Browser text-to-speech
- 📝 Sentence/transcript history
- 📊 Live confidence indicators
- 🦴 Hand-landmark skeleton overlay
- 🔍 Recognition diagnostics for hand count, finger pattern, motion and detected gesture
- 📱 Responsive layout for desktop and mobile browsers
- 🔒 HTTPS camera support through GitHub Pages
- 🚫 No camera frames are uploaded to a project server

## Recognized Signs

The current rule set contains the following 15 signs:

| Sign | Emoji | Recognition approach | Output phrase |
|---|---:|---|---|
| Peace | ✌️ | Index + middle extended | Peace |
| I Love You | 🤟 | Thumb + index + pinky extended | I love you |
| All Good | 👌 | Thumb/index pinch with remaining fingers extended | All good |
| Help | 🆘 | Four fingers extended with thumb tucked, or stable fist near chest | Help |
| Hello | 👋 | Open palm with horizontal movement | Hello |
| Water | 💧 | Index + middle + ring extended; thumb/pinky folded | Please give me water |
| Thanks | 🙏 | Flat hand starting near face and moving outward/down | Thank you very much |
| Friend | 🤝 | Two hands close/interlinked near the chest | You are my friend |
| Stop | ✋ | Open palm held steadily near the chest | Stop, please wait |
| Food / Eat | 🍲 | Pinched/bunched fingertips near the mouth | I want food to eat |
| Yes | 👍 | Thumb-up, optionally with vertical movement | Yes, I agree |
| No | 👎 | Horizontal shaking motion with a non-open hand | No, thank you |
| Cat | 🐱 | Hand near cheek with outward horizontal movement | Cat |
| Please | 🤲 | Open/flat hand making a circular chest movement | Please |
| Sorry | 🙇 | Closed fist making a circular chest movement | I am sorry |

## Recognition Pipeline

    Webcam
      ↓
    MediaPipe Hands
      ↓
    21 hand landmarks
      ↓
    Normalized finger geometry
      ↓
    Hand position + motion history
      ↓
    Temporal gesture rules
      ↓
    Stable sign confirmation
      ↓
    Transcript + speech

## How Recognition Works

The application does not rely only on the current camera frame.

### 1. Hand landmarks

MediaPipe Hands provides 21 landmarks for each detected hand. The application uses these points to estimate finger extension, palm scale, hand position, and relationships between fingertips.

### 2. Normalized finger geometry

Finger-extension checks use joint angles and distances relative to hand/palm scale. This makes the rules less dependent on how close the hand is to the camera.

### 3. Position

The palm center is used to distinguish signs that depend on approximate location:

- **Face region** — signs such as Food, Thanks and Cat
- **Chest region** — signs such as Stop, Please, Sorry and Friend

These regions are approximate screen-space regions; the app does not currently use a face or body-pose model.

### 4. Temporal motion

Movement-based signs are evaluated across a short history of landmark positions rather than from one frame:

- Horizontal movement → Hello / No / Cat
- Vertical movement → Yes
- Face-to-outward movement → Thanks
- Circular chest movement → Please / Sorry

Motion is normalized using the detected hand scale so that the same gesture is less sensitive to camera distance.

### 5. Temporal confirmation

A candidate sign must remain stable for a short period before it is added to the transcript and spoken. A cooldown prevents the same sign from being repeated continuously while the user is holding the pose.

## Recognition Diagnostics

The **Recognition Diagnostics** panel is included to make tuning easier. While the camera is running it shows:

- Hand tracking state
- Number of detected hands
- Five-finger extension pattern
- Whether a pinch is detected
- Normalized horizontal/vertical motion
- Approximate face/chest region
- Current gesture classification and confidence

For example, the finger pattern uses:

    thumb index middle ring pinky
      1     1      1     0     0

This is useful when a sign is not being recognized: first check whether MediaPipe is tracking the hand and whether the detected finger pattern matches the intended pose.

## Running the Project Locally

You can open the HTML through a local web server.

### Python

    git clone https://github.com/Lohithl27/Sign-language.git
    cd Sign-language
    python3 -m http.server 8000

Then open:

    http://localhost:8000

Allow camera access when the browser asks.

> Opening the HTML directly with file:// may prevent webcam access in some browsers. A local HTTP server or the GitHub Pages HTTPS deployment is recommended.

## GitHub Pages Deployment

The repository uses GitHub Actions to deploy the static site.

    Push to main
       ↓
    GitHub Actions
       ↓
    Configure GitHub Pages
       ↓
    Upload repository as Pages artifact
       ↓
    Deploy

The workflow is stored at:

    .github/workflows/pages.yml

GitHub Pages should use **GitHub Actions** as the deployment source.

## Browser Requirements

Recommended:

- Google Chrome
- Microsoft Edge
- Safari on supported devices
- A working webcam
- HTTPS for normal remote camera access
- Internet connection when the page initially loads MediaPipe from the CDN

The app itself does not require a backend API.

## Project Structure

    Sign-language/
    ├── index.html
    ├── README.md
    ├── LICENSE
    └── .github/
        └── workflows/
            └── pages.yml

The current application is intentionally kept as a lightweight single-page web app.

## Important Limitations

This project currently uses a **rule-based heuristic recognizer**, not a trained sign-language recognition model.

In particular:

- Hand position is estimated from the camera frame rather than true face/body landmarks.
- Some signs have visually similar hand configurations.
- Lighting, camera angle, hand rotation, occlusion and background can affect tracking.
- Two-hand signs depend on MediaPipe detecting both hands.
- Dynamic signs require the motion to be performed within the recognition window.
- The 15 signs are predefined gestures rather than a complete sign-language vocabulary.
- Different sign languages and regional signing conventions can use different gestures for the same English word.

For a production-grade system, the recognition layer could be replaced or extended with a trained temporal model using landmark sequences, such as an LSTM/GRU, Transformer, or another sequence-classification architecture.

## Recent Recognition Improvements

The latest recognition update changed the classifier from mainly single-frame rules to a more robust browser-side pipeline:

- Added normalized hand-scale calculations
- Added joint-angle based finger-extension detection
- Added temporal motion history
- Added normalized horizontal/vertical motion measurements
- Added improved two-hand Friend detection
- Added separate handling for dynamic and static gestures
- Added recognition diagnostics
- Improved stable-sign confirmation
- Kept the existing MediaPipe/GitHub Pages architecture
- Kept all processing in the browser

## Development Notes

When adding a new sign:

1. Add the sign definition to SIGNS.
2. Define its static finger pattern.
3. Decide whether it needs hand-position or motion information.
4. Add the rule in classify().
5. Test the sign with different hand sizes, distances and orientations.
6. Use the Recognition Diagnostics panel to inspect the detected landmark state.
7. Update the recognized-sign table in this README.

## Credits

- **MediaPipe Hands** — hand landmark detection and tracking
- **GitHub Pages** — static hosting
- **Web Speech API** — browser speech synthesis

## License

See the repository LICENSE file for the project's license.
