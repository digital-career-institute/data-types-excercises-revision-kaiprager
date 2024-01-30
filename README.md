# java-DB-datatypes

#### 1. Enum data type usage

- Create enum that will contain statuses 'NEW', "PAID", "WAITING FOR PAYMENT".
- Create table "Invoices" with columns: buyer, seller, value, account number, status. Chose the right column types.
- Insert a few rows into this table with different statuses.
- Select only those invoices with status = "NEW"

#### 2. UUID data type usage

- Modify table "Invoices" - add column that will keep "internal_id". 
- Update each row with a value in a column "internal_id". You can use this generator: https://www.uuidgenerator.net/

#### 3. JSON data type usage

- Modify table "Invoices" - add 2 columns that will keep json view of an entity. One column should be with type json and other with type jsonb.
- Try to fill newly created columns with below json:

'{
  "buyer": "company a",
  "seller": "company b",
  "status": "NEW",
  "account_number": 123321123
  "value": 23.43
}



#### 4. Array data type usage

- Modify table "Invoices". Add a column where phone numbers of buyer will be kept. Maximum number of those number is 3.
- Insert a list of phone numbers for an invoice with id = 3: 123211233, 125433221, 127643454
- Insert a list of phone numbers for an invoice with id = 2: 432323112, 123344311
- Select buyer name and his LAST phone number for invoice with id = 3;
- Select an invoice which contains phone: 432323112 

#### 5. Binary data type usage

- Create a file, can be an image or just a text file
- Create a new column in Invoices table that will keep this file
- Insert this file into database for invoice with id = 1


#### 6. Date/time data types usage

- Modify table "Invoices", add 4 new columns: payment deadline, transaction time, transaction hour, cyclic payment. Choose the right column types. 
- Update each column with values
- Update timezone in transaction type, set it to be Melbourne, Australia. 
- Update timezone in transaction time column, set it to be Melbourne, Australia. 
- Select transaction time column, make sure that 10 hours has been added to the time that previously was set.


CREATE TYPE payment_status AS ENUM ('NEW', 'PAID', 'WAITING FOR PAYMENT');

CREATE TABLE Invoices (id SERIAL, buyer VARCHAR(100), seller VARCHAR(100), value DECIMAL(6, 2), account_number INT,
					  status payment_status);
					  
INSERT INTO Invoices (buyer, seller, value, account_number, status) VALUES 
	('Bob', 'Vannessa', 125.56, 56245, 'NEW'),
	('Elfriede', 'Dirk', 6652, 5656546, 'PAID'),
	('John', 'Dieter', 6, 669966, 'WAITING FOR PAYMENT'),
	('Silke', 'Sarah', 885.99, 12554, 'NEW'),
	('Alf', 'Hui Buh', 12.55, 02445, 'PAID'),
	('Dirk', 'Suse', 885.56, 33655, 'WAITING FOR PAYMENT');

SELECT * FROM invoices WHERE status = 'NEW';

ALTER TABLE invoices ADD COLUMN internal_id UUID;

UPDATE invoices SET internal_id = CASE 
    WHEN id = 1 THEN '53002f92-1090-4bdf-98f2-8eedc5d99362'
    WHEN id = 2 THEN '330842e6-99f8-4845-8ba6-744dc30058d9'
    WHEN id = 3 THEN '0715029c-3293-4d83-97e3-eb7ad99fe011'
    WHEN id = 4 THEN '964abec5-2433-43f7-b1fd-ff1004d461c0'
    WHEN id = 5 THEN '95879731-9afd-4431-a8a2-e0197d363c3d'
    WHEN id = 6 THEN 'cb6e5d45-595a-4790-a635-5f1ed1c7fe09'
    ELSE internal_id  
END;

ALTER TABLE invoices ADD COLUMN json_data JSON;

ALTER TABLE invoices ADD COLUMN jason_b_data JSONB;

UPDATE invoices SET json_data =
	'{ "buyer": "company a", "seller": "company b", "status": "NEW", "account_number": 123321123, "value": 23.43 }';

UPDATE invoices SET jason_b_data =
	'{ "buyer": "company a", "seller": "company b", "status": "NEW", "account_number": 123321123, "value": 23.43 }'::jsonb;

ALTER TABLE invoices ADD COLUMN phone_number VARCHAR[3];

UPDATE invoices SET phone_number = ARRAY['123211233', '125433221', '127643454']
	WHERE id = 3;
	
UPDATE invoices SET phone_number = ARRAY['432323112', '123344311']
	WHERE id = 2;

SELECT buyer, phone_number[3] AS last_phone_number FROM invoices WHERE id = 3;

SELECT * FROM invoices WHERE '432323112' = ANY(phone_number);

ALTER TABLE invoices ADD COLUMN image BYTEA;

UPDATE invoices SET image = 
	pg_read_binary_file('/home/dci-student/Pictures/the_park_is_empty_now-rabirius.jpg')::bytea WHERE id = 1;

ALTER TABLE invoices ADD COLUMN payment_deadline DATE,
				ADD COLUMN transaction_time TIMESTAMPTZ,
				ADD COLUMN transaction_hour TIME,
				ADD COLUMN cyclic_payment INTERVAL;

UPDATE invoices SET payment_deadline = '2024-02-24';
UPDATE invoices SET transaction_time = timestamp '2024-01-30 13:48:00';
UPDATE invoices SET transaction_hour = time '11:22:45';
UPDATE invoices SET cyclic_payment = interval '1 MONTH';

UPDATE invoices SET transaction_time = transaction_time AT TIME ZONE 'Australia/Melbourne'
	WHERE id BETWEEN 1 AND 6;

SELECT id, transaction_time FROM invoices;
