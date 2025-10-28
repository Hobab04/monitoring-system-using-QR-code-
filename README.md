php
<?php
session_start();

// Sample role from session
$role = $_SESSION['role'] ?? 'guest';
?>

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Main Menu</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h2>Main Menu</h2>
  <nav>
    <ul>
      <?php if ($role === 'Admin'): ?>
        <li><a href="manage_users.php">Manage Users</a></li>
        <li><a href="generate_reports.php">Generate Reports</a></li>
        <li><a href="settings.php">System Settings</a></li>
      <?php elseif ($role === 'Lecturer'): ?>
        <li><a href="generate_qr.php">Generate QR Code</a></li>
        <li><a href="view_attendance.php">View Attendance</a></li>
        <li><a href="reports.php">Reports</a></li>
      <?php elseif ($role === 'Student'): ?>
        <li><a href="scan_qr.php">Scan QR Code</a></li>
        <li><a href="my_attendance.php">View Attendance</a></li>
      <?php else: ?>
        <li><a href="login.php">Login</a></li>
      <?php endif; ?>
    </ul>
  </nav>
</body>
</html>

javascript
// Highlight active menu item
document.addEventListener("DOMContentLoaded", function() {
  const links = document.querySelectorAll("nav ul li a");
  links.forEach(link => {
    if (link.href === window.location.href) {
      link.classList.add("active");
    }
  });
});

php
<?php
// Simulated data - in real system this comes from DB
$total_students = 1200;
$today_attendance = 875;
$active_courses = 25;
?>

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Control Center</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h2>Control Center</h2>
  <section class="stats">
    <div class="card">Total Students: <?php echo $total_students; ?></div>
    <div class="card">Attendance Today: <?php echo $today_attendance; ?></div>
    <div class="card">Active Courses: <?php echo $active_courses; ?></div>
  </section>

  <section class="quick-links">
    <h3>Quick Access</h3>
    <ul>
      <li><a href="manage_users.php">Manage Users</a></li>
      <li><a href="generate_qr.php">Generate QR</a></li>
      <li><a href="reports.php">View Reports</a></li>
    </ul>
  </section>
</body>
</html>

javascript
// Example: dynamic refresh for attendance count
function refreshAttendance() {
  fetch("get_attendance.php")
    .then(response => response.json())
    .then(data => {
      document.querySelector(".stats .card:nth-child(2)").innerText = 
        "Attendance Today: " + data.attendance;
    })
    .catch(error => console.error("Error:", error));
}

// Auto refresh every 60 seconds
setInterval(refreshAttendance, 60000);


php
<!-- student_management.php -->
<?php
// Fetch student list
include("db_connect.php");
$result = $conn->query("SELECT * FROM students");
?>
<h2>Student Management</h2>
<button onclick="showAddStudentForm()">Add Student</button>
<table border="1">
  <tr><th>ID</th><th>Name</th><th>Matric No</th><th>Action</th></tr>
  <?php while($row = $result->fetch_assoc()): ?>
    <tr>
      <td><?= $row['id']; ?></td>
      <td><?= $row['name']; ?></td>
      <td><?= $row['matric_no']; ?></td>
      <td>
        <button onclick="editStudent(<?= $row['id']; ?>)">Edit</button>
        <button onclick="deleteStudent(<?= $row['id']; ?>)">Delete</button>
      </td>
    </tr>
  <?php endwhile; ?>
</table>

javascript
// student.js
function showAddStudentForm(){
    alert("Open form to add new student");
}
function editStudent(id){
    alert("Edit student with ID: " + id);
}
function deleteStudent(id){
    if(confirm("Are you sure to delete this student?")){
        // AJAX call to backend
    }
}

php
<?php
include("phpqrcode/qrlib.php");
$course_id = 101;
QRcode::png("course:$course_id:".time(), "attendance_qr.png");
echo "<img src='attendance_qr.png' />";
?>

javascript
function generateReport(type){
    alert("Generating " + type + " report...");
    // call PHP backend to fetch and format data
}

PHP (password hashing on registration)
php
<?php
$hashedPassword = password_hash($_POST['password'], PASSWORD_BCRYPT);
$sql = "INSERT INTO users(username, password) VALUES('$username', '$hashedPassword')";
mysqli_query($conn, $sql);
?>
JavaScript (prevent multiple submissions)
javascript
document.getElementById("loginForm").addEventListener("submit", function() {
    document.getElementById("loginBtn").disabled = true;
});

sql
-- Create the database
CREATE DATABASE attendance_system;
USE attendance_system;

-- ===============================
-- 1. USERS TABLE
-- ===============================
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role ENUM('Admin', 'Lecturer', 'Student') NOT NULL
);

-- ===============================
-- 2. STUDENTS TABLE
-- ===============================
CREATE TABLE students (
    student_id INT AUTO_INCREMENT PRIMARY KEY,
    matric_number VARCHAR(20) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(100) NOT NULL,
    level VARCHAR(10) NOT NULL,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- ===============================
-- 3. LECTURERS TABLE
-- ===============================
CREATE TABLE lecturers (
    lecturer_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- ===============================
-- 4. COURSES TABLE
-- ===============================
CREATE TABLE courses (
    course_id INT AUTO_INCREMENT PRIMARY KEY,
    course_code VARCHAR(20) NOT NULL UNIQUE,
    course_title VARCHAR(100) NOT NULL,
    lecturer_id INT,
    FOREIGN KEY (lecturer_id) REFERENCES lecturers(lecturer_id) ON DELETE SET NULL
);

-- ===============================
-- 5. ATTENDANCE TABLE
-- ===============================
CREATE TABLE attendance (
    attendance_id INT AUTO_INCREMENT PRIMARY KEY,
    student_id INT,
    course_id INT,
    date_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('Present', 'Absent') DEFAULT 'Present',
    FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses(course_id) ON DELETE CASCADE
);

-- ===============================
-- 6. QR CODES TABLE
-- ===============================
CREATE TABLE qr_codes (
    qr_id INT AUTO_INCREMENT PRIMARY KEY,
    course_id INT,
    qr_code_data VARCHAR(255) NOT NULL,
    generated_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expiry_time TIMESTAMP,
    FOREIGN KEY (course_id) REFERENCES courses(course_id) ON DELETE CASCADE
);
SQL with Sample Records
sql
-- Insert sample users
INSERT INTO users (username, password, role) VALUES
('admin1', 'admin_pass', 'Admin'),
('lecturer1', 'lecturer_pass', 'Lecturer'),
('student1', 'student_pass', 'Student'),
('student2', 'student_pass', 'Student');

-- Insert sample lecturers
INSERT INTO lecturers (name, department, email, user_id) VALUES
('Dr. John Smith', 'Computer Science', 'jsmith@tsuniversity.edu', 2);

-- Insert sample students
INSERT INTO students (matric_number, name, department, level, user_id) VALUES
('CS2025001', 'Alice Johnson', 'Computer Science', '400', 3),
('CS2025002', 'Michael Brown', 'Computer Science', '400', 4);

-- Insert sample courses
INSERT INTO courses (course_code, course_title, lecturer_id) VALUES
('CSC401', 'Database Systems', 1),
('CSC402', 'Artificial Intelligence', 1);

-- Insert sample QR codes
INSERT INTO qr_codes (course_id, qr_code_data, expiry_time) VALUES
(1, 'QR123ABC', '2025-09-15 23:59:59'),
(2, 'QR456DEF', '2025-09-16 23:59:59');

-- Insert sample attendance
INSERT INTO attendance (student_id, course_id, status) VALUES
(1, 1, 'Present'),
(2, 1, 'Absent'),
(1, 2, 'Present'),
(2, 2, 'Present');

