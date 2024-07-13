# Facial-Recognition-Attendance-System-Database Optimization
My project in Toyota Kirloskar Motors  


CREATE TABLE EmployeeMaster (
    emp_id varchar(50) PRIMARY KEY,
    emp_name varchar(50)
);

-- Insert data into EmployeeMaster
INSERT INTO EmployeeMaster (emp_id, emp_name) VALUES 
('1', 'Amit'),
('2', 'Priya'),
('3', 'Deepika'),
('4', 'Sandeep'),
('5', 'Neha'),
('6', 'Vikram'),
('7', 'Pooja'),
('8', 'Rajesh'),
('9', 'Swati'),
('10', 'Anand');

CREATE TABLE Shift_Master (
    shift_id VARCHAR(3) PRIMARY KEY,
    shift_name VARCHAR(25),
    start_time TIME,
    end_time TIME
);

INSERT INTO Shift_Master (shift_id, shift_name, start_time, end_time)
VALUES
    ('N0G', 'GENERAL', '08:30', '17:30'),
    ('N01', 'Production First', '05:35', '14:45'),
    ('N02', 'Production Second', '14:45', '23:45'),
    ('N0N', 'Production Night', '23:45', '05:35');

CREATE TABLE Shift_Assigned (
    emp_id varchar(50),
    shift_id VARCHAR(3),
    date_assigned DATE,
    FOREIGN KEY (emp_id) REFERENCES EmployeeMaster(emp_id),
    FOREIGN KEY (shift_id) REFERENCES Shift_Master(shift_id)
);


INSERT INTO Shift_Assigned (emp_id, shift_id, date_assigned)
VALUES
    ('1', 'N0G', '2023-11-30'),
    ('2', 'N01', '2023-11-30'),
    ('3', 'N02', '2023-11-30'),
    ('4', 'N0N', '2023-11-30');

    CREATE TABLE RawPunch (
    emp_id varchar(50),
    date DATE,
    punch_time TIME,
    FOREIGN KEY (emp_id) REFERENCES EmployeeMaster(emp_id));


INSERT INTO RawPunch (emp_id, date, punch_time)
VALUES
    ('1', '2023-11-30', '08:27'),
    ('1', '2023-11-30', '08:29'),
    ('1', '2023-11-30', '08:30'),
    ('1', '2023-11-30', '08:30'),
    ('1', '2023-11-30', '09:35'),
    ('1', '2023-11-30', '10:30'),
    ('1', '2023-11-30', '17:29'),
    ('1', '2023-11-30', '17:30'),
    ('1', '2023-11-30', '17:35'),
    ('1', '2023-11-30', '17:36');



INSERT INTO RawPunch (emp_id, date, punch_time)
VALUES
    ('2', '2023-11-30', '05:27'),
    ('2', '2023-11-30', '05:29'),
    ('2', '2023-11-30', '05:30'),
    ('2', '2023-11-30', '05:35'),
    ('2', '2023-11-30', '06:35'),
    ('2', '2023-11-30', '07:30'),
    ('2', '2023-11-30', '14:29'),
    ('2', '2023-11-30', '14:30'),
    ('2', '2023-11-30', '14:35'),
    ('2', '2023-11-30', '14:36');



INSERT INTO RawPunch (emp_id, date, punch_time)
VALUES
    ('3', '2023-11-30', '14:27'),
    ('3', '2023-11-30', '14:29'),
    ('3', '2023-11-30', '14:30'),
    ('3', '2023-11-30', '14:45'),
    ('3', '2023-11-30', '15:35'),
    ('3', '2023-11-30', '15:30'),
    ('3', '2023-11-30', '23:29'),
    ('3', '2023-11-30', '23:30'),
    ('3', '2023-11-30', '23:35'),
    ('3', '2023-11-30', '23:36');



INSERT INTO RawPunch (emp_id, date, punch_time)
VALUES
    ('4', '2023-11-30', '23:27'),
    ('4', '2023-11-30', '23:29'),
    ('4', '2023-11-30', '23:30'),
    ('4', '2023-11-30', '23:45'),
    ('4', '2023-11-30', '00:35'),
    ('4', '2023-12-01', '01:30'),
    ('4', '2023-12-01', '05:29'),
    ('4', '2023-12-01', '05:30'),
    ('4', '2023-12-01', '05:35'),
    ('4', '2023-12-01', '05:36');



WITH PunchSummary AS (
    SELECT
        sa.emp_id,
        sa.shift_id,
        sa.date_assigned,
        MAX(CASE WHEN rp.punch_time < sm.start_time THEN rp.punch_time END) AS checkin_before_shift,
        MIN(CASE WHEN rp.punch_time >= sm.start_time THEN rp.punch_time END) AS checkin_after_scheduled_time,
        MAX(CASE WHEN rp.punch_time > sm.end_time THEN rp.punch_time END) AS checkout_after_shift,
        MAX(rp.punch_time) AS last_punch_within_shift,
        sm.shift_id
    FROM
        RawPunch rp
    JOIN
        Shift_Assigned sa ON rp.emp_id = sa.emp_id AND rp.date = sa.date_assigned
    JOIN
        Shift_Master sm ON sa.shift_id = sm.shift_id
    GROUP BY
        sa.emp_id,
        sa.shift_id,
        sa.date_assigned,
        sm.shift_name
)

SELECT
    ps.emp_id,
    ps.date_assigned AS Date,
    CASE
        WHEN ps.checkin_before_shift IS NOT NULL THEN ps.checkin_before_shift
        ELSE ps.checkin_after_scheduled_time
    END AS check in,
    COALESCE(ps.last_punch_within_shift, ps.checkout_after_shift) AS checkout,
    ps.shift_id
FROM
    PunchSummary ps;


