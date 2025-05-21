# patient-tracking-system
-- Hospital Patient Tracking System Database
-- Purpose: Manage patients, doctors, appointments, and treatments
-- Features: Normalized schema, sample data, and analytical queries

-- 1. Database Schema Creation
--------------------------------

-- Create and use database
DROP DATABASE IF EXISTS occ;
CREATE DATABASE occ;
USE occ;

-- Patients table
DROP TABLE IF EXISTS patient;
CREATE TABLE patient (
    patient_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    date_of_birth DATE,
    gender CHAR(1) CHECK (gender IN ('M', 'F', 'O')),
    phone VARCHAR(15),
    email VARCHAR(100),
    address TEXT,
    blood_type VARCHAR(3),
    registration_date DATE DEFAULT (CURRENT_DATE)
);

-- Doctors table
DROP TABLE IF EXISTS doctor;
CREATE TABLE doctor (
    doctor_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    specialization VARCHAR(100),
    phone VARCHAR(15),
    email VARCHAR(100),
    hire_date DATE,
    department VARCHAR(50)
);

-- Treatments table
DROP TABLE IF EXISTS treatment;
CREATE TABLE treatment (
    treatment_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    cost DECIMAL(10, 2) CHECK (cost >= 0)
);

-- Appointments table
DROP TABLE IF EXISTS appointment;
CREATE TABLE appointment (
    appointment_id INT PRIMARY KEY,
    patient_id INT,
    doctor_id INT,
    appointment_date TIMESTAMP NOT NULL,
    status VARCHAR(20) NOT NULL CHECK (status IN ('Scheduled', 'Completed', 'Cancelled', 'No-Show')),
    reason TEXT,
    diagnosis TEXT,
    prescription TEXT,
    FOREIGN KEY (patient_id) REFERENCES patient(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES doctor(doctor_id)
);

-- Patient treatments (junction table)
DROP TABLE IF EXISTS patient_treatment;
CREATE TABLE patient_treatment (
    treatment_record_id INT PRIMARY KEY,
    patient_id INT,
    treatment_id INT,
    doctor_id INT,
    treatment_date DATE NOT NULL,
    notes TEXT,
    FOREIGN KEY (patient_id) REFERENCES patient(patient_id),
    FOREIGN KEY (treatment_id) REFERENCES treatment(treatment_id),
    FOREIGN KEY (doctor_id) REFERENCES doctor(doctor_id)
);

-- 2. Sample Data Insertion
---------------------------

-- Insert sample patient
INSERT INTO patient VALUES
(1, 'Charita', 'Pandey', '1985-03-15', 'F', '555-0101', 'charita.pandey22@email.com', '402 Main road', 'A+', '2023-01-10'),
(2, 'Anisha', 'Sharma', '1990-07-22', 'F', '555-0102', 'anishasharmaaa@email.com', '20/2 Gandhi nagar', 'B-', '2023-02-15');

-- Insert sample doctors
INSERT INTO doctor VALUES
(101, 'Sara', 'Chetan', 'Cardiology', '555-0201', 's.chet@hospital.com', '2018-05-10', 'Cardiology'),
(102, 'Mohan', 'Singh', 'Neurology', '555-0202', 'm.singh@hospital.com', '2020-03-15', 'Neurology');

-- Insert sample treatments
INSERT INTO treatment VALUES
(501, 'ECG', 'Electrocardiogram', 120.00),
(502, 'MRI Scan', 'Magnetic Resonance Imaging', 450.00);

-- Insert sample appointments
INSERT INTO appointment VALUES
(1001, 1, 101, '2023-10-15 09:00:00', 'Completed', 'Chest pain', 'Mild arrhythmia', 'Beta blockers'),
(1002, 2, 102, '2023-10-16 10:30:00', 'Scheduled', 'Headaches', NULL, NULL);

-- Insert sample treatments
INSERT INTO patient_treatment VALUES
(1, 1, 501, 101, '2023-10-15', 'Normal sinus rhythm with occasional PVCs'),
(2, 2, 502, 102, '2023-10-17', 'Scheduled for MRI evaluation');

-- 3. Analytical Queries
------------------------

-- Query 1: Find all completed appointments with diagnosis and treatment
SELECT 
    p.patient_id,
    CONCAT(p.first_name, ' ', p.last_name) AS patient_name,
    CONCAT(d.first_name, ' ', d.last_name) AS doctor_name,
    a.appointment_date,
    a.diagnosis,
    t.name AS treatment,
    pt.treatment_date
FROM 
    appointment a
JOIN 
    patient p ON a.patient_id = p.patient_id
JOIN 
    doctor d ON a.doctor_id = d.doctor_id
LEFT JOIN 
    patient_treatment pt ON a.patient_id = pt.patient_id AND a.doctor_id = pt.doctor_id
LEFT JOIN 
    treatment t ON pt.treatment_id = t.treatment_id
WHERE 
    a.status = 'Completed'
ORDER BY 
    a.appointment_date DESC;

-- Query 2: Doctor workload analysis
SELECT 
    d.doctor_id,
    CONCAT(d.first_name, ' ', d.last_name) AS doctor_name,
    d.specialization,
    COUNT(DISTINCT a.appointment_id) AS total_appointments,
    COUNT(DISTINCT pt.patient_id) AS patients_treated,
    COALESCE(SUM(t.cost), 0) AS total_revenue_generated
FROM 
    doctor d
LEFT JOIN 
    appointment a ON d.doctor_id = a.doctor_id
LEFT JOIN 
    patient_treatment pt ON d.doctor_id = pt.doctor_id
LEFT JOIN 
    treatment t ON pt.treatment_id = t.treatment_id
GROUP BY 
    d.doctor_id, d.first_name, d.last_name, d.specialization
ORDER BY 
    total_appointments DESC;
