# Dr. Omega App

Dr. Omega is a cutting-edge web application designed to assist doctors in identifying diseases from uploaded reports (PDF, JPG, PNG, etc.). The app provides detailed disease descriptions, suggested medicines, and precautions. The project integrates a FastAPI backend with a Next.js frontend to deliver a robust, scalable, and user-friendly experience.

## Features

- **File Upload**: Supports multiple file formats like PDF, JPG, and PNG.
- **Disease Identification**: Utilizes LangChain and LLMs for analyzing uploaded reports.
- **Chat Support**: Interactive chat interface to answer queries related to the uploaded reports.
- **Data Analytics**: Statistical insights for research and studies.
- **Responsive Design**: Tailored for devices of all sizes.

## Tech Stack

### Backend
- **Framework**: FastAPI
- **Libraries**: LangChain, FAISS, Pydantic
- **Database**: PostgreSQL
- **Utilities**: File upload handling, CORS middleware
- **Environment**: Poetry for dependency management

### Frontend
- **Framework**: Next.js
- **Libraries**: React, Axios
- **Styling**: TailwindCSS
- **Components**: Reusable UI components for scalability

## Installation

### Prerequisites

- Python 3.8+
- Node.js 14+
- npm or yarn
- Docker

### Backend Setup

1. Clone the repository and navigate to the `backend/` directory:
   ```bash
   git clone https://github.com/your-repo/dr-omega.git
   cd dr-omega/backend
   ```
   **Why?**: This ensures you have the complete source code for the backend and that you are working in the correct directory.

2. Install Poetry:
   ```bash
   pip install poetry
   ```
   **Why?**: Poetry is a dependency manager for Python projects. It simplifies installation and ensures all required packages are managed efficiently.

3. Use Poetry to install dependencies:
   ```bash
   poetry install
   ```
   **Why?**: This command reads the `pyproject.toml` file and installs all necessary dependencies for the backend application.

4. Start the FastAPI server with Poetry:
   ```bash
   poetry run uvicorn app.main:app --reload
   ```
   **Why?**: `uvicorn` is the server used to run FastAPI. The `--reload` flag allows automatic reloading during development whenever code changes.

### Frontend Setup

1. Navigate to the `frontend/` directory:
   ```bash
   cd ../frontend
   ```
   **Why?**: This step positions you in the frontend folder where the Next.js application is located.

2. Build and run the frontend Docker container:
   ```bash
   docker build -t dr-omega-frontend .
   docker run -d -p 3000:3000 --name dr-omega-frontend-container dr-omega-frontend
   ```
   **Why?**: This builds and starts the frontend application in a Docker container, ensuring consistency across development environments.

## Project Structure

### Backend
```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                # FastAPI entry point
│   ├── api/
│   │   ├── endpoints.py       # API endpoints (e.g., /chat, /upload)
│   ├── core/
│   │   ├── config.py          # Configuration settings (e.g., CORS, constants)
│   │   ├── langchain.py       # LangChain initialization and utilities
│   ├── models/
│   │   ├── schemas.py         # Pydantic models for request/response validation
│   ├── utils/
│       ├── file_utils.py      # Utilities for handling file uploads
├── uploads/                   # Directory for storing uploaded files
├── Dockerfile                 # Docker configuration for backend
├── pyproject.toml             # Poetry configuration for dependencies
├── README.md                  # Project documentation
└── .gitignore                 # Git ignored files
```

### Frontend
```
frontend/
├── public/                     # Public assets (images, icons, etc.)
│   ├── favicon.ico
│   ├── logo.png
├── src/
│   ├── components/             # Reusable UI components
│   │   ├── ChatComponent.tsx   # Chat UI logic
│   │   ├── MessageBox.tsx      # Individual chat message box
│   │   ├── ModeToggle.tsx      # Theme toggle button
│   │   ├── ThemeProvider.tsx   # Theme provider
│   ├── pages/                  # Pages for the application
│   │   ├── index.tsx           # Homepage
│   │   ├── api/                # Next.js API routes (if any)
│   ├── styles/                 # Tailwind CSS or other styling
│   │   ├── globals.css         # Global styles
│   ├── utils/                  # Utility functions
│       ├── apiClient.ts        # Axios or fetch wrapper for API calls
├── Dockerfile                  # Docker configuration for frontend
├── .env                        # Environment variables
├── .gitignore                  # Git ignored files
├── next.config.js              # Next.js configuration
├── package.json                # Node.js dependencies
└── tsconfig.json               # TypeScript configuration
```

## Dockerfile for Backend

**Path:** `backend/Dockerfile`
```dockerfile
# Use official Python image as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container
COPY . /app

# Install Poetry
RUN pip install poetry

# Install dependencies using Poetry
RUN poetry install

# Make port 8000 available to the world outside this container
EXPOSE 8000

# Run FastAPI server
CMD ["poetry", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Dockerfile for Frontend

**Path:** `frontend/Dockerfile`
```dockerfile
# Use official Node.js image as a parent image
FROM node:16-alpine

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container
COPY . /app

# Install any needed packages
RUN npm install

# Build the frontend application
RUN npm run build

# Make port 3000 available to the world outside this container
EXPOSE 3000

# Run Next.js server
CMD ["npm", "start"]
```

## API Endpoints

### Backend Endpoints

1. **Upload File**: `/upload/`
   - Method: POST
   - Description: Upload medical reports for analysis.

   **Code:**
   ```python
   @app.post("/upload/")
   async def upload_file(file: UploadFile = File(...)):
       """Endpoint to handle file uploads."""
       file_location = os.path.join(UPLOAD_DIR, file.filename)
       try:
           with open(file_location, "wb") as f:
               shutil.copyfileobj(file.file, f)
           return {"message": f"File '{file.filename}' uploaded successfully", "file_path": file_location}
       except Exception as e:
           raise HTTPException(status_code=500, detail=f"Failed to upload file: {e}")
   ```

2. **Chat**: `/chat/`
   - Method: POST
   - Description: Engage in a conversation to understand the disease details.

   **Code:**
   ```python
   @app.post("/chat/")
   async def chat(message: str = Form(...), history: str = Form("[]")):
       """Endpoint to process chat interactions."""
       try:
           chat_history = json.loads(history)
           response = qa_chain.run({"query": message, "chat_history": chat_history})
           return {"response": response}
       except Exception as e:
           raise HTTPException(status_code=500, detail=f"Chat processing failed: {e}")
   ```

3. **Status**: `/status/`
   - Method: GET
   - Description: Check server health.

   **Code:**
   ```python
   @app.get("/status/")
   async def status():
       """Health check endpoint."""
       return {"status": "Running", "service": "Dr. Omega"}
   ```

### Frontend Pages

1. **Home**: `/`
   - Description: Landing page for the application.

2. **Chat Interface**: `/chat`
   - Description: Chat with the AI assistant.

   **Code:**
   ```tsx
   import React, { useState } from "react";
   import axios from "axios";
   import MessageBox from "./MessageBox";

   type Props = {
     reportData: string;
   };

   const ChatComponent = ({ reportData }: Props) => {
     const [messages, setMessages] = useState<{ role: string; content: string }[]>([]);
     const [input, setInput] = useState("");

     const handleSubmit = async (e: React.FormEvent) => {
       e.preventDefault();
       const response = await axios.post("http://localhost:8000/chat/", {
         message: input,
         history: JSON.stringify(messages),
       });
       setMessages([...messages, { role: "user", content: input }, { role: "assistant", content: response.data.response }]);
       setInput("");
     };

     return (
       <div className="chat-container">
         <div className="messages">
           {messages.map((msg, idx) => (
             <MessageBox key={idx} role={msg.role} content={msg.content} />
           ))}
         </div>
         <form onSubmit={handleSubmit} className="chat-form">
           <input
             type="text"
             value={input}
             onChange={(e) => setInput(e.target.value)}
             placeholder="Type your message..."
           />
           <button type="submit">Send</button>
         </form>
       </div>
     );
   };

   export default ChatComponent;
   ```

## Contributing

Feel free to fork this repository and submit pull requests. For major changes, please open an issue first to discuss what you would like to change.

## License

This project is licensed under the [MIT License](LICENSE).

## Acknowledgments

- [FastAPI](https://fastapi.tiangolo.com/)
- [Next.js](https://nextjs.org/)
- [LangChain](https://www.langchain.com/)
- [OpenAI](https://openai.com/)
