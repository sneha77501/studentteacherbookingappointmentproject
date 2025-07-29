# studentteacherbookingappointmentproject
index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Appointment System</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h1>Student-Teacher Booking Appointment </h1>

  <!-- Admin Panel -->
  <section>
    <h2>Admin Panel - Add Teacher</h2>
    <input id="adminTeacherName" placeholder="Name" />
    <input id="adminTeacherDept" placeholder="Department" />
    <input id="adminTeacherSubject" placeholder="Subject" />
    <input id="adminTeacherEmail" placeholder="Email" />
    <button onclick="addTeacher()">Add Teacher</button>
    <h3>All Teachers:</h3>
    <div id="teacherList"></div>
  </section>

  <!-- Teacher Panel -->
  <section>
    <h2>Teacher Panel</h2>
    <input id="teacherLoginEmail" placeholder="Login Email" />
    <button onclick="loginTeacher()">Login</button>
    <button onclick="viewAppointments()">View Appointments</button>
    <div id="teacherAppointments"></div>
  </section>

  <!-- Student Panel -->
  <section>
    <h2>Student Panel</h2>
    <input id="studentName" placeholder="Name" />
    <input id="studentEmail" placeholder="Email" />
    <button onclick="registerStudent()">Register</button>

    <h3>Search Teacher by Subject:</h3>
      <input type="text" id="searchSubject" placeholder="Enter subject" />
      <button onclick="searchTeacher()">Search</button>
      <div id="searchResults"></div>


    <h3>Book Appointment</h3>
    <input id="appointmentTeacherId" placeholder="Teacher ID" />
    <input id="appointmentTime" placeholder="Date & Time" />
    <button onclick="bookAppointment()">Book</button>
  </section>

  <script type="module" src="firebase-config.js"></script>
  <script type="module" src="script.js"></script>
</body>
</html>
 firebase-config.js
 // Firebase SDK imports
import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-app.js";
import { getDatabase, ref, set, push, onValue } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-database.js";

// Your Firebase config
const firebaseConfig = {
  apiKey: "AIzaSyASjKsVhQNGbk_sEdCoOMFott5K8zpEuys",
  authDomain: "student-teacher-appointm-248a4.firebaseapp.com",
  databaseURL: "https://student-teacher-appointm-248a4-default-rtdb.firebaseio.com",
  projectId: "student-teacher-appointm-248a4",
  storageBucket: "student-teacher-appointm-248a4.appspot.com",
  messagingSenderId: "930599499854",
  appId: "1:930599499854:web:fbcf8476f731aa0491eb1f"
};

// Initialize
const app = initializeApp(firebaseConfig);
const db = getDatabase(app);

export { db, ref, set, push, onValue };

import { db, ref, set, push, onValue } from "./firebase-config.js";

// Add Teacher (Admin)
window.addTeacher = function () {
  const name = document.getElementById("adminTeacherName").value;
  const department = document.getElementById("adminTeacherDept").value;
  const subject = document.getElementById("adminTeacherSubject").value;
  const email = document.getElementById("adminTeacherEmail").value;

  if (!name || !department || !subject || !email) {
    alert("Please fill all teacher details");
    return;
  }

  const id = push(ref(db, "teachers")).key;
  set(ref(db, "teachers/" + id), {
    id,
    name,
    department,
    subject,
    email
  }).then(() => {
    alert("Teacher added successfully");
    loadTeachers();
  });
};

// Load Teachers (Admin)
function loadTeachers() {
  const container = document.getElementById("teacherList");
  container.innerHTML = "Loading...";

  onValue(ref(db, "teachers"), (snapshot) => {
    const data = snapshot.val();
    if (!data) return container.innerHTML = "No teachers found.";

    container.innerHTML = Object.values(data).map(t => `
      <div>
        <b>${t.name}</b> - ${t.subject} (${t.department})<br>
        ID: ${t.id}<br><hr>
      </div>
    `).join("");
  });
}
loadTeachers();

// Student Registration
window.registerStudent = function () {
  const name = document.getElementById("studentName").value;
  const email = document.getElementById("studentEmail").value;

  if (!name || !email) {
    alert("Enter name and email");
    return;
  }

  const id = push(ref(db, "students")).key;
  set(ref(db, "students/" + id), {
    id,
    name,
    email
  }).then(() => alert("Student registered successfully"));
};

// Search Teacher



window.searchTeacher = function () {
  const subject = document.getElementById("searchSubject").value.trim().toLowerCase();
  const resultsDiv = document.getElementById("searchResults");
  resultsDiv.innerHTML = "Searching...";

  const teacherRef = ref(db, "teachers");
  onValue(teacherRef, (snapshot) => {
    const data = snapshot.val();
    resultsDiv.innerHTML = ""; // Clear previous results
    let found = false;

    for (let key in data) {
      const teacher = data[key];
      if (teacher.subject && teacher.subject.toLowerCase().includes(subject)) {
        resultsDiv.innerHTML += `
          <div class="teacher-result">
            <h4>${teacher.name}</h4>
            <p><b>Subject:</b> ${teacher.subject}</p>
            <p><b>Email:</b> ${teacher.email}</p>
          </div><hr/>
        `;
        found = true;
      }
    }

    if (!found) {
      resultsDiv.innerHTML = "No matching teachers found.";
    }
  }, (error) => {
    console.error("Search failed:", error);
    resultsDiv.innerHTML = "Error loading teachers.";
  });
};


// Book Appointment (Student)
window.bookAppointment = function () {
  const teacherId = document.getElementById("appointmentTeacherId").value;
  const time = document.getElementById("appointmentTime").value;

  if (!teacherId || !time) {
    alert("Enter teacher ID and time");
    return;
  }

  const id = push(ref(db, "appointments")).key;
  set(ref(db, "appointments/" + id), {
    id,
    teacherId,
    time,
    status: "pending"
  }).then(() => alert("Appointment booked"));
};

// Teacher Login
window.loginTeacher = function () {
  const email = document.getElementById("teacherLoginEmail").value.trim().toLowerCase(); // Normalize email

  const teacherRef = ref(db, "teachers");
  onValue(teacherRef, (snapshot) => {
    const data = snapshot.val();

    if (!data) {
      alert("No teachers found.");
      return;
    }

    let matched = null;
    for (const id in data) {
      if (data[id].email && data[id].email.toLowerCase() === email) {
        matched = { id, ...data[id] };
        break;
      }
    }

    if (matched) {
      localStorage.setItem("teacherId", matched.id);
      alert("Logged in successfully!");
      // Optionally redirect or load appointments
    } else {
      alert("Teacher not found. Please check email.");
    }
  }, (error) => {
    console.error("Error reading teacher data:", error);
    alert("Login failed due to database error.");
  });
};


// View Appointments (Teacher)


window.viewAppointments = function () {
  const teacherId = localStorage.getItem("teacherId");
  const container = document.getElementById("appointmentList");

  if (!teacherId) {
    alert("You must be logged in as a teacher.");
    return;
  }

  const appointmentRef = ref(db, "appointments");

  onValue(appointmentRef, (snapshot) => {
    const data = snapshot.val();
    container.innerHTML = "";

    if (!data) {
      container.innerHTML = "No appointments found.";
      return;
    }

    let html = "";
    let hasData = false;

    Object.values(data).forEach(app => {
      if (app.teacherId === teacherId) {
        html += `
          <div class="appointment-card">
            <p><b>Student:</b> ${app.studentName}</p>
            <p><b>Subject:</b> ${app.subject}</p>
            <p><b>Date:</b> ${app.date}</p>
            <p><b>Status:</b> ${app.status || "Pending"}</p>
            <hr/>
          </div>
        `;
        hasData = true;
      }
    });

    container.innerHTML = hasData ? html : "No appointments for you.";
  }, (error) => {
    console.error("Error fetching appointments:", error);
    container.innerHTML = "Failed to load appointments.";
  });
};

body {
  font-family: sans-serif;
  margin: 0;
  padding: 20px;
  background-color: #a5e9af;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
}

section {
  background-color: white;
  padding: 20px;
  margin: 10px auto;
  max-width: 600px;
  border-radius: 8px;
  box-shadow: 0 0 10px #ccc;
}

input {
  width: 100%;
  padding: 10px;
  margin: 6px 0;
  border-radius: 4px;
  border: 1px solid #ccc;
}

button {
  background-color: #007BFF;
  color: white;
  padding: 10px 15px;
  border: none;
  margin-top: 8px;
  cursor: pointer;
  border-radius: 4px;
}

button:hover {
  background-color: #0056b3;
}

#teacherList, #searchResults, #teacherAppointments {
  background: #f9f9f9;
  padding: 10px;
  border-radius: 5px;
  margin-top: 10px;
}

