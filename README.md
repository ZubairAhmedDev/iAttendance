# iAttendance

iAttendance is a web-based attendance management system that uses face recognition to register employees and record employee IN/OUT attendance. The project combines an ASP.NET MVC web application with a Python Flask API for face registration and face matching.

## Overview

The system allows an employee/user to:

- Register employee details with a face image
- Capture an attendance image from the web application
- Match the captured face against registered face data
- Automatically mark attendance as IN or OUT
- Store employee and attendance records in a MySQL database

## Features

- User login page
- Employee face registration
- Webcam/image-based attendance capture
- Face recognition using Python
- Automatic IN/OUT status handling
- Attendance logging with employee number, employee name, time, status, and device ID
- MySQL database integration
- ASP.NET MVC frontend with Python backend API integration

## Tech Stack

### Frontend / Web Application

- ASP.NET MVC
- C#
- Razor Views
- HTML
- CSS
- JavaScript

### Backend / Services

- ASP.NET MVC Controllers and Services
- Python Flask API
- `face_recognition` Python library
- MySQL database

### Database

- MySQL

## Project Structure

```text
iAttendance/
│
├── App_Start/                 # ASP.NET MVC route and bundle configuration
├── Content/                   # CSS and static content
├── Controllers/               # MVC controllers
│   ├── AttendanceController.cs # Handles attendance image capture and face match request
│   ├── EmployeeController.cs   # Employee-related controller logic
│   ├── HomeController.cs       # Main/home page routing
│   ├── Login.cs                # Login controller
│   ├── RegistrationController.cs # Handles employee registration
│   └── InOutController.cs      # Attendance report controller work
│
├── Models/                    # C# model classes
├── Properties/                # Project properties
├── Scripts/                   # JavaScript files
├── Services/                  # C# service/database/API helper classes
│   ├── Employee.cs            # Employee database helper
│   ├── LoginDB.cs             # Login database helper
│   ├── RegistrationConfig.cs  # Calls Python API for face registration
│   ├── RegistrationDB.cs      # Registration database helper
│   ├── WebConfig.cs           # Calls Python API for face matching
│   └── InOut.cs               # Attendance report service logic
│
├── Views/                     # Razor view pages
│
├── iAttendenceP/              # Python Flask face-recognition API
│   ├── app.py                 # Flask API entry point
│   ├── RegisterFaces.py       # Registers face encodings
│   ├── FaceMatch.py           # Matches captured face with saved encodings
│   ├── Db.py                  # MySQL attendance logging logic
│   └── IP.py                  # IP-related helper file
│
├── Global.asax
├── Web.config
├── iAttendance.csproj
├── iAttendance.sln
└── packages.config
```

## How the Application Works

### 1. Login

The user logs in through the ASP.NET MVC login page. Login credentials are checked against the `log_in` table in the MySQL database.

### 2. Employee Registration

During registration:

1. The employee number, employee name, and face image are submitted from the web application.
2. The ASP.NET MVC registration controller saves the image.
3. The C# service sends the image file name and employee details to the Python Flask `/RegisterFace` API.
4. The Python API extracts the face encoding using the `face_recognition` library.
5. The encoding is saved into a `.dat` file.
6. Employee details are also stored in the MySQL `register` table.

### 3. Attendance Capture

During attendance marking:

1. The user captures an image from the web application.
2. The ASP.NET MVC attendance controller saves the captured image.
3. The C# service sends the image details to the Python Flask `/FaceMatch` API.
4. The Python API compares the captured face with registered face encodings.
5. If a match is found, attendance is logged in MySQL.
6. The system automatically toggles the employee status:
   - If the employee was last OUT, the new entry becomes IN.
   - If the employee was last IN, the new entry becomes OUT.

## Python API Endpoints

### Register Face

```http
POST /RegisterFace
```

Registers a new employee face.

Expected JSON body:

```json
{
  "filename": "employee_image.png",
  "employeeNo": "101",
  "empName": "John Doe"
}
```

### Face Match

```http
POST /FaceMatch
```

Matches a captured attendance image with registered face data.

Expected JSON body:

```json
{
  "imgFileName": "captured_image.png",
  "deviceId": "DEVICE-001"
}
```

## Database Tables

The project expects a MySQL database named:

```text
iattendance
```

### Suggested Tables

#### `log_in`

Stores login credentials.

```sql
CREATE TABLE log_in (
    USER_NAME VARCHAR(100) NOT NULL,
    PASSWORD VARCHAR(100) NOT NULL
);
```

#### `register`

Stores registered employee details.

```sql
CREATE TABLE register (
    EMP_NO INT PRIMARY KEY,
    EMP_NAME VARCHAR(150) NOT NULL,
    IMG LONGTEXT
);
```

#### `Attendance`

Stores attendance IN/OUT records.

```sql
CREATE TABLE Attendance (
    ID INT AUTO_INCREMENT PRIMARY KEY,
    EMP_NO INT NOT NULL,
    EMP_NAME VARCHAR(150) NOT NULL,
    IN_TIME DATETIME,
    OUT_TIME DATETIME,
    IN_OUT_STATUS TINYINT,
    DEVICE_ID VARCHAR(255)
);
```

## Prerequisites

Before running the project, install the following:

- Visual Studio
- .NET Framework 4.7.2
- MySQL Server
- Python 3.x
- pip
- Git

Python packages:

```bash
pip install flask face-recognition numpy mysql-connector-python
```

> Note: The `face_recognition` package may require additional system dependencies such as `dlib`.

## Configuration

Update the ASP.NET `Web.config` file with your MySQL connection string and Python API URL.

Example:

```xml
<connectionStrings>
  <add name="DBConnection"
       connectionString="server=localhost;database=iattendance;user=root;password=your_password;"
       providerName="MySql.Data.MySqlClient" />

  <add name="RDBConnection"
       connectionString="server=localhost;database=iattendance;user=root;password=your_password;"
       providerName="MySql.Data.MySqlClient" />
</connectionStrings>

<appSettings>
  <add key="APIServerName" value="http://127.0.0.1:5000/" />
  <add key="ImageDirectory" value="C:\Websites\iAttendance\Registration" />
  <add key="CropFileLocation" value="C:\Websites\iAttendance\CapturedImage" />
</appSettings>
```

Also update the MySQL settings inside:

```text
iAttendenceP/Db.py
```

Example:

```python
connection_config = {
    "host": "localhost",
    "database": "iattendance",
    "user": "root",
    "password": "your_password",
}
```

## How to Run the Project

### Step 1: Clone the Repository

```bash
git clone https://github.com/ZubairAhmedDev/iAttendance.git
cd iAttendance
```

### Step 2: Set Up the Database

1. Create a MySQL database named `iattendance`.
2. Create the required tables.
3. Add at least one login user in the `log_in` table.

Example:

```sql
INSERT INTO log_in (USER_NAME, PASSWORD)
VALUES ('admin', 'admin123');
```

### Step 3: Start the Python Flask API

Go to the Python API folder:

```bash
cd iAttendenceP
python app.py
```

The Flask API should run locally, usually at:

```text
http://127.0.0.1:5000/
```

### Step 4: Run the ASP.NET MVC Project

1. Open `iAttendance.sln` in Visual Studio.
2. Restore NuGet packages.
3. Build the solution.
4. Run the project using IIS Express.

## Important Notes

- Make sure the Python Flask API is running before using face registration or attendance capture.
- Make sure the folder paths in `Web.config` match the actual folders on your machine.
- Make sure MySQL credentials are correct in both the ASP.NET configuration and Python `Db.py`.
- The current project uses plain text passwords. For production use, passwords should be hashed.
- The current project uses local file paths such as `C:\Websites\iAttendance\...`. These should be changed based on your local setup.

## Future Improvements

- Add password hashing for login security
- Add role-based access for admin and employee users
- Add better error messages on the frontend
- Add attendance report filtering by date
- Add export to Excel/PDF
- Move hardcoded paths and database credentials into environment variables
- Add duplicate employee registration validation
- Add dashboard charts for attendance summary
- Improve UI design and responsiveness
- Add deployment instructions

## Author

**Zubair Ahmed**

GitHub: [ZubairAhmedDev](https://github.com/ZubairAhmedDev)


**Nida Minhaj**

GitHub: [Nidaa26](https://github.com/Nidaa26?tab=overview&from=2026-02-01&to=2026-02-28)
