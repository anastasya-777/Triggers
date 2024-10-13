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



2. Создать триггер, который фиксирует в журнале сведений о преподавателях все операции манипулирования, про- изводимые над таблицей преподавателей.

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
        * INSERT INTO TeacherManipulations (Date, ActionId, TeacherId)
        * SELECT GETDATE(), 3, d.Id
        * FROM deleted d;
    * END
* END;

### Скриншот результата

<img width="857" alt="image" src="https://github.com/user-attachments/assets/fa655a04-2cf0-4949-82d8-eb81442f556d">



