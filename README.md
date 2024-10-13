# Triggers

## Теория баз данных

## Тема: Триггеры.

### Задание

1. Создать триггер, который позволяет только увеличивать размер фонда финансирования факультета.

### Скрипт запроса
* CREATE TRIGGER trg_validate_salary
* ON Teachers
* INSTEAD OF UPDATE
* AS
* BEGIN
    -- Проверяем, не уменьшается ли зарплата
    * IF EXISTS (
        * SELECT 1
        * FROM inserted i
        * JOIN deleted d ON i.Id = d.Id
        * WHERE i.Salary < d.Salary)
    * BEGIN
        * RAISERROR ('Зарплата преподавателя может только увеличиваться.', 16, 1);
        * ROLLBACK TRANSACTION;
    * END
    * ELSE
    * BEGIN
        -- Если зарплата не уменьшается, выполняем обновление
        * UPDATE Teachers
        * SET EmploymentDate = i.EmploymentDate,
            * Name = i.Name,
            * Surname = i.Surname,
            * Salary = i.Salary
        * FROM inserted i
        * WHERE Teachers.Id = i.Id;
    * END
* END;

### Скриншот результата

<img width="820" alt="image" src="https://github.com/user-attachments/assets/aec7835d-f887-42b7-a07f-c68f8f1e547f">


1.1 При попытке поставить заработную плату ниже предыдушей, то будет появляться предупреждение о недопустимости данного действия:

### Скрипт запроса с изменением зарплаты

* UPDATE Teachers SET Salary = 55000 WHERE Name = 'Сергей';

### Скриншот результата
<img width="806" alt="image" src="https://github.com/user-attachments/assets/5676b716-5586-4231-9459-f86cc53b0d5f">




2. Создать триггер, который фиксирует в журнале сведений о преподавателях все операции манипулирования, производимые над таблицей преподавателей.

### Скрипт запроса

* CREATE TRIGGER trg_log_teacher_changes
* ON Teachers
* AFTER INSERT, UPDATE, DELETE
* AS
* BEGIN -- Логирование вставки IF EXISTS (SELECT * FROM inserted) AND NOT EXISTS (SELECT * FROM deleted)
    * BEGIN
        * INSERT INTO TeacherManipulations (Date, ActionId, TeacherId)
        * SELECT GETDATE(), 1, i.Id
        * FROM inserted i;
    * END

    -- Логирование обновления IF EXISTS (SELECT * FROM inserted) AND EXISTS (SELECT * FROM deleted)
    * BEGIN
        * INSERT INTO TeacherManipulations (Date, ActionId, TeacherId)
        * SELECT GETDATE(), 2, i.Id
        * FROM inserted i;
    * END

    -- Логирование удаления
    * IF EXISTS (SELECT * FROM deleted) AND NOT EXISTS (SELECT * FROM inserted)
    * BEGIN
        -- Обработка нескольких удаленных записей INSERT INTO TeacherManipulations (Date, ActionId, TeacherId)
        * SELECT GETDATE(), 3, d.Id
        * FROM deleted d;
    * END
* END;

### Скриншот результата

<img width="920" alt="image" src="https://github.com/user-attachments/assets/092edf53-a25b-40b8-9cc1-a6859be54710">

2.1 Добавим нового преподавателя и проверим, что в таблице TeacherManipulations появится запись с ActionId = 1.

### Скрипт для добавления 

* INSERT INTO Teachers (Name, Surname, Salary, EmploymentDate)
* VALUES ('Анна', 'Антонова', 50000, '2023-04-01');

### Скрипт для проверки 

* SELECT * FROM TeacherManipulations WHERE TeacherId = (SELECT Id FROM Teachers WHERE Name = 'Анна');

### Скриншот результата

<img width="895" alt="image" src="https://github.com/user-attachments/assets/016547f0-5093-4680-aad3-09a76ebabfa9">

2.2 Обновим зарплату существующего преподавателя, например, Ивана Иванова, и проверим, что в таблице TeacherManipulations появится запись с ActionId = 2.

### Скрипт обновления 

* UPDATE Teachers SET Salary = 60000 WHERE Name = 'Иван';

### Скрипт для проверки

* SELECT * FROM TeacherManipulations WHERE TeacherId = (SELECT Id FROM Teachers WHERE Name = 'Иван');

### Скриншот результата

<img width="836" alt="image" src="https://github.com/user-attachments/assets/996674bc-7aab-47de-9362-c5c0f0583812">


2.3 Удалим преподавателя, например, Петра Петрова, и проверим, что в таблице TeacherManipulations появится запись с ActionId = 3.

### Скрипт для удаления

* DELETE FROM TeacherManipulations WHERE TeacherId = (SELECT Id FROM Teachers WHERE Name = 'Петр');
* DELETE FROM Teachers WHERE Name = 'Петр';

### Скрипт для проверки

* SELECT * FROM TeacherManipulations WHERE TeacherId = 2;

### Скриншот результата

<img width="779" alt="image" src="https://github.com/user-attachments/assets/33d4ee74-acc4-4c31-be45-8b03cc887588">



  





