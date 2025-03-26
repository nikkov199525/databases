# Лабораторная работа 2

Ковалев Никита  
Группа: p4150  
Дата выполнения задания: 24.03.2025  
Именование дисциплины: Взаимодействие с базами данных  

## Текст задания:
1.  Из описания предметной области, полученной в ходе выполнения ЛР 1, выделить сущности, их атрибуты и связи, отразить их в инфологической модели (она же концептуальная)
2.  Составить даталогическую (она же ER-диаграмма, она же диаграмма сущность-связь) модель. При описании типов данных для атрибутов должны использоваться типы из СУБД PostgreSQL.
3.  Реализовать даталогическую модель в PostgreSQL. При описании и реализации даталогической модели должны учитываться ограничения целостности, которые характерны для полученной предметной области
4.  Заполнить созданные таблицы тестовыми данными.
Для построения моделей можно использовать plantUML





## Описание предметной области: Поликлиника
В поликлинике работают врачи различных специальностей. Каждый день в поликлинику обращаются пациенты.
Все пациенты проходят обязательную регистрацию, при которой в базу данных заносятся стандартные анкетные данные (фамилия, имя, отчество, год рождения). Каждый пациент может обращаться в поликлинику несколько раз, нуждаясь в различной медицинской помощи.
Все обращения больных фиксируются, при этом устанавливается диагноз, назначается Лечение, фиксируется дата обращения.
При обращении в поликлинику пациент может проходить лечение у разных специалистов.
## Инфологическая модель
Основные сущности и их атрибуты:

- Специальности (Specialties)
  - specialty_id (первичный ключ)
  - specialty_name

- Врачи (Doctors)
  - doctor_id (первичный ключ)
  - surname
  - name
  - middlename
  - cabinet_number
  - specialty_id (внешний ключ)

- Пациенты (Patients)
  - med_card_id (первичный ключ)
  - surname
  - name
  - middlename
  - address
  - phone
  - workplace

- Записи на прием (Appointments)
  - appointment_id (первичный ключ)
  - med_card_id (внешний ключ)
  - doctor_id (внешний ключ)
  - appointment_date
  - diagnos
  - treatment

Связи:
- Специальности ↔ Врачи один ко многим
- Врачи ↔ Записи на прием один ко многим
-Пациенты ↔ Записи на прием один ко многим
## Даталогическая модель
Даталогическая модель (ER-диаграмма) описана в формате PlantUML.

@startuml
entity "Specialties" as specialties {
  + specialty_id : SERIAL <<PK>>
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
## Реализация даталогической модели в в PostgreSQL и заполннение таблиц данными
Реализацию даталогической модели и заполнение базы данными можно выполнить при помощи следующего скрипта:

CREATE TABLE Specialties (
    specialty_id SERIAL PRIMARY KEY,
    specialty_name VARCHAR(100) NOT NULL
);

CREATE TABLE Doctors (
    doctor_id SERIAL PRIMARY KEY,
    surname VARCHAR(100) NOT NULL,
    firstname VARCHAR(100) NOT NULL,
    middlename VARCHAR(100) NOT NULL,
    cabinet_number INT NOT NULL,
    specialty_id INT NOT NULL, 
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

INSERT INTO Specialties (specialty_name) VALUES
('Терапевт'),
('Уролог'),
('Гинеколог'),
('Хирург');

INSERT INTO Doctors (surname, firstname, middlename, cabinet_number, specialty_id) VALUES                                                        
('Власов', 'Герман', 'Иванович', 201, 1),                                 
('Крестовский', 'Петр', 'Сергеевич', 204, 4);

INSERT INTO Patients (surname, name, middle_name, address, phone, workplace) VALUES                                                        
('Капранов', 'Дмитрий', 'Евгеньевич', 'Санкт-Петербург, ул. Пушкина, д. 6', '+79244789631', 'АО Яндекс');

INSERT INTO Appointments (med_card_id, doctor_id, appointment_date, diagnos, treatment) VALUES
(1, 1, '2025-01-20', 'Covid19', 'противовирусные препараты, постельный режим');



## Результат выполнения
Для того, чтобы увидеть результат выполнения скрипта, приведенного в предыдущем разделе, были выполнены простейшие запросы:
select * from Specialties;
select * from Doctors;
select * from Patients;
select * from Appointments;
Результаты выполнения запросов приведены ниже.


 specialty_id | specialty_name 
--------------+----------------
            1 | Терапевт
            2 | Уролог
            3 | Гинеколог
            4 | Хирург
(4 строки)

 doctor_id |   surname   | firstname | middlename | cabinet_number | specialty_id 
-----------+-------------+-----------+------------+----------------+--------------
         1 | Власов      | Герман    | Иванович   |            201 |            1
         2 | Крестовский | Петр      | Сергеевич  |            204 |            4
(2 строки)

 med_card_id | surname  |  name   | middle_name |              address               |    phone     | workplace 
-------------+----------+---------+-------------+------------------------------------+--------------+-----------
           1 | Капранов | Дмитрий | Евгеньевич  | Санкт-Петербург, ул. Пушкина, д. 6 | +79244789631 | АО Яндекс
(1 строка)

 appointment_id | med_card_id | doctor_id | appointment_date | diagnos |                  treatment                  
----------------+-------------+-----------+------------------+---------+---------------------------------------------
              1 |           1 |         1 | 2025-01-20       | Covid19 | противовирусные препараты, постельный режим
(1 строка)


