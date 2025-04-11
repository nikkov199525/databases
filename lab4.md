# Лабораторная работа 4

Ковалев Никита  
Группа: p4150  
Дата выполнения задания: 08.04.2025  
Именование дисциплины: Взаимодействие с базами данных  
## Текст задания

·  Описать бизнес-правила вашей предметной области. Какие в вашей системе могут быть действия, требующие выполнения запроса в БД. Эти бизнес-правила будут использованы для реализации триггеров, функций, процедур, транзакций поэтому приниматься будут только достаточно интересные бизнес-правила
·  Добавить в ранее созданную базу данных триггеры для обеспечения комплексных ограничений целостности. Триггеров должно быть не менее трех
·  Реализовать функции и процедуры на основе описания бизнес-процессов, определенных при описании предметной области из пункта 1. Примеров не менее 3
·  Привести 3 примера выполнения транзакции. Это может быть, например, проверка корректности вводимых данных для созданных функций и процедур. Например, функция, которая вносит данные. Данные проверяются и в случае если они не подходят ограничениям целостности, транзакция должна откатываться
·  Необходимо произвести анализ использования созданной базы данных, выявить наиболее часто используемые объекты базы данных, виды запросов к ним. Результаты должны быть представлены в виде текстового описания
·  На основании полученного описания требуется создать подходящие индексы и доказать, что они будут полезны для представленных в описании случаев использования базы данных.

### Описание бизнес-правил

Регистрация пациентов: каждый пациент должен пройти регистрацию, где должны быть занесены обязательные анкетные данные. Наличие дубликатов по ФИО не допускается.
Запись на прием: пациент не может быть записан на прием к одному и тому же врачу дважды.
Корректность записи на прием: дата приема пациента не может быть раньше текущей даты.

## Функции, процедуры и триггеры

### Процедура для добавления нового приема

CREATE OR REPLACE PROCEDURE add_appointment(patient_id INT, doctor_id INT, appointment_date DATE, diagnos VARCHAR, treatment TEXT) AS $$
BEGIN
    INSERT INTO Appointments (med_card_id, doctor_id, appointment_date, diagnos, treatment) 
    VALUES (patient_id, doctor_id, appointment_date, diagnos, treatment);
END;
$$ LANGUAGE plpgsql;

### Предотвращение дубликатов пациентов
#### Функция
CREATE OR REPLACE FUNCTION prevent_duplicate_patients() 
RETURNS TRIGGER AS $$
BEGIN
    IF EXISTS (SELECT 1 FROM Patients WHERE surname = NEW.surname AND name = NEW.name AND middle_name = NEW.middle_name) THEN
        RAISE EXCEPTION 'Пациент с такими ФИО уже зарегистрирован.';
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
#### Триггер
CREATE TRIGGER trg_prevent_duplicate_patients
BEFORE INSERT ON Patients
FOR EACH ROW EXECUTE FUNCTION prevent_duplicate_patients();

### ограничение количества записей на прием
#### Функция
CREATE OR REPLACE FUNCTION limit_appointments_per_time() 
RETURNS TRIGGER AS $$
BEGIN
    IF EXISTS (
        SELECT 1 FROM Appointments 
        WHERE med_card_id = NEW.med_card_id 
          AND doctor_id = NEW.doctor_id 
          AND appointment_date = NEW.appointment_date
    ) THEN
        RAISE EXCEPTION 'Пациент уже записан на прием к данному врачу в указанное время.';
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
#### Триггер
CREATE TRIGGER trg_limit_appointments_per_time
BEFORE INSERT ON Appointments
FOR EACH ROW EXECUTE FUNCTION limit_appointments_per_time();

### Проверка корректности даты приема
#### Функция
CREATE OR REPLACE FUNCTION check_appointment_date() 
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.appointment_date < CURRENT_DATE THEN
        RAISE EXCEPTION 'Дата приема не может быть раньше текущей.';
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
#### Триггер
CREATE TRIGGER trg_check_appointment_date
BEFORE INSERT ON Appointments
FOR EACH ROW EXECUTE FUNCTION check_appointment_date();

### Демонстрация работы транзакций
#### Добавление пациента
BEGIN;
INSERT INTO Patients (surname, name, middle_name, address, phone, workplace) VALUES ('Иванов', 'Сергей', 'Алексеевич', 'Москва, ул. Ленина, д. 10', '+74212245676', 'АО "Верное решение"');
COMMIT;
#### Запись на прием к врачу
BEGIN;
INSERT INTO Appointments (med_card_id, doctor_id, appointment_date, diagnos, treatment) VALUES (1, 1, '2025-01-21', 'Грипп', 'Постельный режим');
COMMIT;
#### Ошибка при добавлении записи
BEGIN;
INSERT INTO Appointments (med_card_id, doctor_id, appointment_date, diagnos, treatment) VALUES (1, 1, '2025-01-20', 'Кашель', 'Противовирусные препараты'); 
COMMIT;
### Создание индексов
В ходе анализа применения базы данных были выявлены наиболее часто используемые объекты:
Таблица Patients;
Таблица Doctors.
Исходя из этого были созданы следующие индексы:
Индекс в таблице Patients ускоряют выполнение запросов по имени пациента.
Индекс в таблице  Doctors ускоряет поиск врачей по специальностям, что полезно при регистрации пациентов и их обращениях.
#### Индекс на таблицу Patients
CREATE INDEX idx_patient_name ON Patients (surname, name, middle_name);
#### Индекс на таблицу Doctors
CREATE INDEX idx_doctor_specialty ON Doctors (specialty_id);
