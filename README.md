# Smart Parking Management System

## Overview
Automated parking lot management system using Raspberry Pi, OCR camera, and RFID sensors to control vehicle entry and exit.

## Data Structure

### ParkingLog Model
**Purpose**: Record vehicle entry history (entry log only)

**Note**: This model only stores information when a vehicle **enters**, not when it exits. When a vehicle exits, the system will:
- Find the record by `cardId`
- Compare license plates
- Calculate parking duration
- Delete/mark the record as processed

**Fields**:
- **licensePlate**: Vehicle license plate (required, automatically converted to uppercase)
- **entryTime**: Vehicle entry time (required, defaults to current time)
- **cardId**: Vehicle card/RFID ID (required)
- **image**: Photo of vehicle on entry (optional)

## System Workflow

### 1. Entry Lane Flow
**Equipment**: Raspberry Pi + Camera + RFID Reader

**Process**:
1. Vehicle arrives at entry gate
2. Camera captures vehicle photo
3. OCR recognizes license plate
4. RFID Reader reads vehicle card ID
5. Record entry time
6. Save data to MongoDB (licensePlate, entryTime, cardId, image)
7. Open gate for vehicle entry

### 2. Exit Lane Flow
**Equipment**: Raspberry Pi + Camera + RFID Reader

**Process**:
1. Vehicle arrives at exit gate
2. RFID Reader reads vehicle card ID
3. Camera captures exit photo
4. OCR recognizes exit license plate
5. Record exit time (automatically generated when processing starts)
6. Send data to system: `cardId`, `exitLicensePlate`, `exitImage`, `exitTime`

**Validation**:
- Query database by `cardId`
- Compare entry license plate (from database) with exit license plate (from OCR)
- **If match**: 
  - Calculate parking duration (exitTime - entryTime)
  - Display comparison of entry/exit images
  - Open exit gate
  - Delete record from database
  - Display information: license plate, card, entry/exit time, parking duration
- **If no match**: 
  - Alert license plate mismatch (display both plates)
  - Display exit image for verification
  - Do not open gate
  - Log the incident

**Input Data (from Raspberry Pi)**:
- `cardId`: Card ID from RFID Reader (required)
- `exitLicensePlate`: License plate from OCR (required)
- `exitImage`: Exit image URL (optional)
- `exitTime`: Exit time (automatically generated during processing)

### 3. Parking Duration Calculation
```
Parking Duration = Exit Time - entryTime (from database)
```

## Technologies Used

### Backend
- **Node.js + Express**: API server
- **MongoDB + Mongoose**: Database
- **Socket.io**: Real-time communication (if needed)

### Frontend
- **React + Vite**: Management interface
- **TailwindCSS**: Styling

### Raspberry Pi
- **Python**: Camera and GPIO processing
- **OpenCV**: Image processing
- **OCR**: License plate recognition
- **MFRC522/RC522**: RFID Reader

## Project Structure
```
parking/
├── controller/        # API controllers
├── model/            # MongoDB models
├── frontend/         # React frontend
├── raspberry-pi/     # Python scripts for Raspberry Pi
│   ├── camera_ocr_service.py  # Camera and OCR processing
│   ├── gpio_control.py        # GPIO/gate control
│   └── main.py               # Main script
└── utils/           # Utilities and middleware
```

## API Endpoints (Planned)

### Entry
- `POST /api/parking/entry` - Record vehicle entry
  - Body: `{ licensePlate, cardId, image, entryTime }`

### Exit
- **Frontend Service: `processExit(cardId, exitLicensePlate)`**
  - Find vehicle by `cardId`
  - Validate matching license plate
  - Delete record if valid
  - Additional inputs: `exitImage` (URL), `exitTime` (auto-generated)
  - Response: Entry/exit vehicle info, parking duration, status, comparison images

### Query
- `GET /api/parking/logs` - Get log list
- `GET /api/parking/current` - Currently parked vehicles
- `GET /api/parking/card/:cardId` - Query by card

## Installation Requirements

### Backend
```bash
npm install
```

### Frontend
```bash
cd frontend
npm install
```

### Raspberry Pi
```bash
pip install opencv-python pytesseract mfrc522
```

## Running the Application

### Backend
```bash
npm start
```

### Frontend
```bash
cd frontend
npm run dev
```

### Raspberry Pi
```bash
cd raspberry-pi
python main.py
```

## Security Notes
- Authenticate RFID card before processing
- Log all incidents (license plate mismatches)
- Regular database backups
- Encrypt sensitive data if needed

## Future Features
- [ ] Automatic parking fee calculation
- [ ] Email/SMS notifications
- [ ] Analytics dashboard
- [ ] Webhook API for external systems
- [ ] Multi-language support