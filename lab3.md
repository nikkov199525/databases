# Лабораторная работа 3

Ковалев Никита  
Группа: p4150  
Дата выполнения задания: 22.03.2025  
Именование дисциплины: Взаимодействие с базами данных  

## Текст задания:
Для выполнения лабораторной работы №3 необходимо:
1.  Реализованную в рамках лабораторной работы №2 даталогическую модель привести в 3 нормальную форму. Если в вашей предметной области эффективнее использовать денормализованную модель – это нужно обосновать при сдаче ЛР. 
2.  Привести 3 примера анализа функциональной зависимости атрибутов. Соответственно, для трех таблиц (в предметной области каждого студента должно быть не менее трех таблиц). 
3.  Обеспечить целостность данных таблиц при помощи средств языка DDL. Задача – это продемонстрировать знание видов ограничений целостности. Чем больше будет использовано, тем меньше вопросов будет задано. 
4.  Заполнить таблицы данными
5.  В рамках лабораторной работы должны быть разработаны скрипты-примеры для создания/удаления требуемых объектов базы данных, заполнения/удаления содержимого созданных таблиц. Студент должен быть готов продемонстрировать работу скриптов!
6.  Составить 6+3 примеров SQL запросов на объединение таблиц предметной области. Студент должен быть готов продемонстрировать работу запросов и обосновать их результаты, почему они получились именно такими. 6 – это INNER, FULL, LEFT, RIGTH, CROSS, OUTER и еще 3 JOIN ON, JOIN USING, NATURAL JOIN


## Описание предметной области: Поликлиника
В поликлинике работают врачи различных специальностей. Каждый день в поликлинику обращаются пациенты. Все пациенты проходят обязательную регистрацию, при которой в базу данных заносятся стандартные анкетные данные (фамилия, имя, отчество, адрес, телефон и место работы). Каждый пациент может обращаться в поликлинику несколько раз, нуждаясь в различной медицинской помощи. Все обращения фиксируются с указанием диагноза и лечения. Пациенты могут проходить лечение у разных врачей.


## Даталогическая модель
Даталогическая модель (ER-диаграмма) описана в формате PlantUML.

@startuml
entity "Specialties" as specialties {
  + specialty_id : VARCHAR(50) <<PK>>
  + specialty_name : VARCHAR(100)
}

entity "Doctors" as doctors {
  + doctor_id : SERIAL <<PK>>
  + surname : VARCHAR(50)
  + name : VARCHAR(50)
  + middlename : VARCHAR(50)
  + cabinet_number : INT
  + specialty_id : INT <<FK>>
}

entity "Patients" as patients {
  + med_card_id : SERIAL <<PK>>
  + surname : VARCHAR(50)
  + name : VARCHAR(50)
  + middlename : VARCHAR(50)
  + address : VARCHAR(150)
  + phone : VARCHAR(12)
  + workplace : VARCHAR(200)
}

entity "Appointments" as appointments {
  + appointment_id : INT <<PK>>
  + med_card_id : INT <<FK>>
  + doctor_id : INT <<FK>>
  + appointment_date : DATE
  + diagnos : VARCHAR(150)
  + treatment : TEXT
}
specialties ||--o{ doctors : ""
doctors ||--o{ appointments : ""
patients ||--o{ appointments : ""

@enduml

## Функциональная зависимость
Таблица Doctors
   - doctor_id → surname, firstname, middlename, cabinet_number, specialty_id

Таблица Patients
   - med_card_id → surname, name, middle_name

Таблица Appointments
appointment_id → med_card_id, doctor_id, appointment_date, diagnos, treatment






## Обеспечение целостности данных
Для обеспечения целостности данных в запрос, создающий таблицы, были внесены некоторые изменения. Обновленный запрос приводится ниже.

CREATE TABLE Specialties (
    specialty_id VARCHAR(50) PRIMARY KEY CHECK (specialty_id ~ '^[A-Za-z0-9_]+$'),
    specialty_name VARCHAR(100) NOT NULL
);

CREATE TABLE Doctors (
    doctor_id SERIAL PRIMARY KEY,
    surname VARCHAR(100) NOT NULL,
    firstname VARCHAR(100) NOT NULL,
    middlename VARCHAR(100) NOT NULL,
    cabinet_number INT NOT NULL,
    specialty_id VARCHAR(50) NOT NULL, 
    FOREIGN KEY (specialty_id) REFERENCES Specialties(specialty_id)
);

CREATE TABLE Patients (
    med_card_id SERIAL PRIMARY KEY,
    surname VARCHAR(50) NOT NULL,
    name VARCHAR(50) NOT NULL,
    middle_name VARCHAR(50) NOT NULL,
    address VARCHAR(150) NOT NULL,
    phone VARCHAR(12),
    workplace VARCHAR(200)
);

CREATE TABLE Appointments (
    appointment_id SERIAL PRIMARY KEY,
    med_card_id INT NOT NULL,
    doctor_id INT NOT NULL,
    appointment_date DATE NOT NULL,
    diagnos VARCHAR(150) NOT NULL,
    treatment TEXT,
    FOREIGN KEY (med_card_id) REFERENCES Patients(med_card_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctors(doctor_id)
);






## Примеры заполнения данных каждой таблицы

### Заполнение таблицы Specialties
INSERT INTO Specialties (specialty_id, specialty_name) VALUES
('terapeft', 'Терапевт'),
('urolog', 'Уролог'),
('gynecolog', 'Гинеколог'),
('herurg', 'Хирург');


### Заполнение таблицы Doctors

INSERT INTO Doctors (surname, firstname, middlename, cabinet_number, specialty_id) VALUES                                                        
('Власов', 'Герман', 'Иванович', 201, 'terapeft'),                                 
('Крестовский', 'Петр', 'Сергеевич', 204, 'herurg');

### Заполнение таблицы Patients
INSERT INTO Patients (surname, name, middle_name,address, phone, workplace)
VALUES ('Капранов', 'Дмитрий', 'Евгеньевич', 1, 'Санкт-Петербург, ул. Пушкина, д. 6', '+79244789631', 'АО Яндекс');


### Заполнение таблицы Appointments
INSERT INTO Appointments (med_card_id, doctor_id, appointment_date, diagnos, treatment) 
VALUES (1, 1, '2025-01-20', 'Кашель', 'Противовирусные препараты');

## Пример запроса на обновление


UPDATE Doctors
SET cabinet_number = 202
WHERE doctor_id = 1;

## Пример Запроса на удаление

DELETE FROM Patients
WHERE med_card_id = 1;
## Запросы на объединение таблиц (Join)

INNER JOIN

   SELECT Patients.surname AS patient_surname, Doctors.surname AS doctor_surname
   FROM Patients
   INNER JOIN Appointments ON Patients.med_card_id = Appointments.med_card_id
   INNER JOIN Doctors ON Appointments.doctor_id = Doctors.doctor_id;
возвращает фамилии пациентов и врачей, которые имеют хотя бы одну запись о приеме.

LEFT JOIN

   SELECT Patients.surname AS patient_surname, Appointments.appointment_date
   FROM Patients
   LEFT JOIN Appointments ON Patients.med_card_id = Appointments.med_card_id;
возвращает фамилии всех пациентов и даты их приемов, если такие имеются.


RIGHT JOIN

   SELECT Doctors.surname AS doctor_surname, Appointments.appointment_date
   FROM Doctors
   RIGHT JOIN Appointments ON Doctors.doctor_id = Appointments.doctor_id;
возвращает фамилии всех врачей и даты их приемов, включая приемы, которые могут не иметь соответствующего врача.

FULL OUTER JOIN

   SELECT Patients.surname AS patient_surname, Doctors.surname AS doctor_surname
   FROM Patients
   FULL OUTER JOIN Appointments ON Patients.med_card_id = Appointments.med_card_id
   FULL OUTER JOIN Doctors ON Appointments.doctor_id = Doctors.doctor_id;
возвращает фамилии всех пациентов и врачей, независимо от того, есть ли у них приёмы.

CROSS JOIN

   SELECT Patients.surname AS patient_surname, Doctors.surname AS doctor_surname
   FROM Patients
   CROSS JOIN Doctors;
возвращает каждую возможную комбинацию пациента и врача.


JOIN ON

   SELECT Patients.surname AS patient_surname, Appointments.appointment_date
   FROM Patients
   JOIN Appointments ON Patients.med_card_id = Appointments.med_card_id;
возвращает фамилии пациентов и даты их приемов, при условии, что такие приемы существуют.


JOIN USING

   SELECT Patients.surname AS patient_surname, Appointments.appointment_date
   FROM Patients
   JOIN Appointments USING (med_card_id);
То же самое, что и предыдущий запрос, но с использованием USING.



NATURAL JOIN

   SELECT Patients.surname AS patient_surname, Doctors.surname AS doctor_surname
   FROM Patients
   NATURAL JOIN Appointments
   NATURAL JOIN Doctors;
возвращает фамилии пациентов и врачей, соединив их по общим полям.
