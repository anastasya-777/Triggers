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
