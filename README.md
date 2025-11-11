# Standard-Code

## S3 Code 
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-bucket-name/*"
        }
    ]
}
```
## ec2 with auto scale 
Scroll down and expand the Advanced details section for user data field.
```
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from Fat-Lab! My IP is $(hostname -f)</h1>" > /var/www/html/index.html
```

## lambda + S3 
```
import json

def lambda_handler(event, context):
    # The 'event' variable contains all the information about the trigger.
    # We parse it to get the bucket name and the file name (key).
    
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # We use a simple print() statement. In Lambda, this prints to the logs.
    print(f"Success! A file named '{key}' was uploaded to the S3 bucket '{bucket}'.")
    
    return {
        'statusCode': 200,
        'body': json.dumps('Log generated successfully!')
    }
```


## SQL 
```
-- Step 1: Create and use the database
CREATE DATABASE company;
USE company;

-- Step 2: Create the tables
-- Table to store employee information
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    position VARCHAR(100),
    salary DECIMAL(10, 2)
);

-- Table to act as an audit log for salary changes
CREATE TABLE salary_audits (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    employee_id INT,
    old_salary DECIMAL(10, 2),
    new_salary DECIMAL(10, 2),
    change_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (employee_id) REFERENCES employees(id)
);

-- Step 3: Insert some initial data
INSERT INTO employees (name, position, salary) VALUES
('Alice', 'Software Engineer', 70000.00),
('Bob', 'Project Manager', 85000.00);

-- Step 4: Create the Trigger
-- This trigger automatically logs a record into salary_audits whenever an employee's salary is updated.
DELIMITER $$
CREATE TRIGGER before_employee_salary_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    -- Check if the salary value is actually changing
    IF OLD.salary <> NEW.salary THEN
        INSERT INTO salary_audits(employee_id, old_salary, new_salary)
        VALUES(OLD.id, OLD.salary, NEW.salary);
    END IF;
END$$
DELIMITER ;

-- Step 5: Create the Stored Procedure
-- This procedure gives a specified employee a raise by a certain percentage.
DELIMITER $$
CREATE PROCEDURE give_raise(IN emp_id INT, IN raise_percentage DECIMAL(5,2))
BEGIN
    UPDATE employees
    SET salary = salary * (1 + raise_percentage / 100)
    WHERE id = emp_id;
END$$
DELIMITER ;
```
