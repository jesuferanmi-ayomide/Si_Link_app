# Si-LINK: System Architecture

This document outlines the technical architecture for the Si-LINK mobile application.

## 1. Technology Stack

### Frontend (Mobile App)
* **Framework:** **Flutter**
* **Why:** Flutter allows us to build a single, high-performance application for both iOS and Android from one codebase. Its direct access to device hardware (like the camera) is essential for the sign language recognition feature.

### Backend (API & AI Services)
* **Framework:** **Python (FastAPI)**
* **Why:** Python is the industry standard for machine learning (TensorFlow, PyTorch, OpenCV). FastAPI is a modern, high-performance web framework that is easy to build with and can handle asynchronous tasks, which is perfect for serving our AI models and managing real-time connections.

* **AI / Machine Learning:**
    * **Sign-to-Text Model:** A custom-trained **TensorFlow** or **PyTorch** model (likely a CNN + RNN/Transformer) that will be hosted as a microservice.
    * **Speech-to-Text Service:** We will leverage a third-party API like **Google Cloud Speech-to-Text** for high-accuracy, real-time transcription.
    * **Text-to-Sign Animation:** This will likely use a pre-built library of sign-language animations, mapped from the text output.

### Database
* **Database:** **PostgreSQL**
* **Why:** We need a reliable, relational database to store user-profiles, settings, and potentially a library of saved/common phrases. PostgreSQL is powerful, scalable, and integrates well with Python.

## 2. Component Communication Flow

The system is designed as a set of services communicating via REST APIs and WebSockets.

![A simple diagram showing the flow: Mobile App communicates with the Main API. The Main API communicates with the User Database, the Speech-to-Text API, and the Sign-to-Text AI Model.](https://miro.com/app/board/uXjVJxTeN9k=/?share_link_id=757869119784)
**Flow 1: Sign-to-Text (User 1 is Signing)**
1.  **Mobile App (Flutter):** Captures video frames from the camera.
2.  **Mobile App (Flutter):** Streams these frames (or processed keypoints) to the backend.
3.  **Backend (FastAPI):** Receives the stream and passes it to the **Sign-to-Text AI Model**.
4.  **AI Model (TensorFlow):** Interprets the signs and returns a text string (e.g., "Hello, how are you?").
5.  **Backend (FastAPI):** Pushes this text string to the other user's app via a **WebSocket**.
6.  **Mobile App (Flutter):** The hearing user's app receives the text and converts it to speech using the device's native Text-to-Speech (TTS) engine.

**Flow 2: Speech-to-Text (User 2 is Speaking)**
1.  **Mobile App (Flutter):** Captures audio from the microphone.
2.  **Mobile App (Flutter):** Streams this audio to the **Google Speech-to-Text API**.
3.  **Speech-to-Text API:** Returns a text string (e.g., "I'm good, thanks!").
4.  **Mobile App (Flutter):** Sends this text string to the **Backend (FastAPI)**.
5.  **Backend (FastAPI):** Pushes this text to the signing user's app via a **WebSocket**.
6.  **Mobile App (Flutter):** The deaf user's app receives the text and either displays it clearly or renders the corresponding **Sign Language Animation**.

## 3. Technical Feasibility

This architecture is highly feasible.
* **Core Challenge:** The primary challenge is not the architecture, but the **accuracy of the sign language recognition model**. This will require a significant amount of high-quality, labeled video data for training.
* **Scalability:** The architecture is scalable. The AI models can be deployed as independent microservices and scaled separately from the main API (e.g., using Kubernetes or a serverless platform).
* **Real-time:** Using WebSockets for communication between users and third-party APIs for speech-to-text ensures the "real-time" feel of the conversation.
* **Cross-Platform:** Using Flutter solves the challenge of building for both iOS and Android simultaneously, which is critical for a communication tool.
