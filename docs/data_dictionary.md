# data_dictionary

## таблиця users

для акаунтів користувачів, зберігає їхні персональні дані

| Field | Type | Constraint | Description |
| --- | --- | --- | --- |
| id | INT | PK, AUTO_INCREMENT | унікальний ідентифікатор користувача |
| email | VARCHAR | NOT NULL, UNIQUE | ел. пошта користувача для входу |
| password_hash | VARCHAR | NOT NULL | пароль з хешуванням для входу |
| full_name | VARCHAR | NOT NULL | ім’я та прізвище користувача |
| age | INT | NOT NULL | вік користувача |
| sex | ENUM('male', 'female') | NOT NULL | стать гравця для поділу на категорії |
| skill_level | ENUM('beginner', 'intermediate', 'advanced', 'professional') | NOT NULL | рівень гри користувача: початківець/середній/вище середнього/професійний |
| playing_hand | ENUM('left', 'right') | NOT NULL | домінуюча рука у грі: ліва/права |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | час, коли було створено акаунт |

## таблиця user_roles

зв’язувальна таблиця, надає роль користувачу

| Field | Type | Constraint | Description |
| --- | --- | --- | --- |
| id | INT | PK, AUTO_INCREMENT | унікальний ідентифікатор ролі |
| user_id | INT | NOT NULL, FK > users.id | надання користувачу ролі |
| role | ENUM('player', 'organizer') | NOT NULL | назва ролі: гравець/організатор |
| (user_id, role) | | UNIQUE | забезпечення того, що користувач не отримає повторно ту саму роль |

## таблиця tournaments

описує один турнір з бадмінтону

| Field | Type | Constraint | Description |
| --- | --- | --- | --- |
| id | INT | PK, AUTO_INCREMENT | унікальний ідентифікатор турніру |
| name | VARCHAR | NOT NULL | офіційна назва турніру |
| description | TEXT | NULL | опціональний детальний опис турніру |
| start_date | DATE | NOT NULL | дата початку турніру |
| end_date | DATE | NOT NULL | дата закінчення турніру |
| location | VARCHAR | NULL | місце проведення турніру |
| status | ENUM('upcoming', 'ongoing', 'completed', 'cancelled') | DEFAULT 'upcoming' | поточний статус турніру: ще не почато/в процесі/завершено/скасовано |
| created_by_user_id | INT | NOT NULL, FK > users.id | організатор, який створив турнір |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | час створення турніру |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | час останнього оновлення інформації |
| max_sets | TINYINT | DEFAULT 3 | максимальна кількість сетів (зазвичай 3) |
| points_to_win | INT | NOT NULL, DEFAULT 21 | необхідна кількість очок для виграшу сету (зазвичай 21) |
| win_by_two | TINYINT(1) | NOT NULL, DEFAULT 1 | TRUE, якщо встановили правило “до різниці в 2 очки” для уникнення нічиї |
| max_points | INT | DEFAULT 30 | максимальна кількість очок за сет, якщо встановлено правило “до різниці в 2 очки” (зазвичай 30) |

## таблиця tournament_participants

зв’язувальна таблиця між гравцями і турнірами до яких вони доєдналися. Гравці самі можуть доєднуватися до доступного турніру, але лише організатору дозволено ставити одного гравця проти іншого (одиночна гра). Зокрема, таблиця слугує джерелом гравців для парних ігор, команди для яких також створює організатор

| Field | Type | Constraint | Description |
| --- | --- | --- | --- |
| id | INT | PK, AUTO_INCREMENT | унікальний ідентифікатор для учасника турніру |
| tournament_id | INT | NOT NULL, FK > tournaments.id | в якому турнірі бере участь гравець |
| user_id | INT | NOT NULL, FK > users.id | гравець, який доєднався до турніру |
| registration_date | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | час коли гравець доєднався до турніру |
| (tournament_id, user_id) | | UNIQUE | запобігає додаванню користувача до одного й того самого турніру |

## таблиця teams

описує команду для парної гри в турнірі (1 команда - 2 гравця)

| Field | Type | Constraint | Description |
| --- | --- | --- | --- |
| id | INT | PK, AUTO_INCREMENT | унікальний ідентифікатор команди |
| tournament_id | INT | NOT NULL, FK > tournaments.id | частиною якого турніру є команда |
| player1_id | INT | NOT NULL, FK > users.id | перший учасник команди |
| player2_id | INT | NOT NULL, FK > users.id | другий учасник команди |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | час коли було створено команду |

## таблиця matches

описує один матч (одиночна або парна гра) всередині турніру

| Field | Type | Constraint | Description |
| --- | --- | --- | --- |
| id | INT | PK, AUTO_INCREMENT | унікальний ідентифікатор матчу |
| tournament_id | INT | NOT NULL, FK > tournaments.id | до якого турніру належить матч |
| match_category | ENUM('MS', 'WS', 'MD', 'WD', 'XD') | NOT NULL | категорія матчу: чоловіча одиночна/жіноча одиночна/чоловіча парна/жіноча парна/змішана парна гра |
| match_date | DATETIME | NOT NULL | планова дата і час матчу |
| winner_player_id | INT | NULL, FK > users.id | гравець, що виграв, якщо match_category = ‘MS’ або ‘WS’. має NULL допоки матч не завершиться |
| winner_team_id | INT | NULL, FK > teams.id | команда, що виграла, якщо match_category = ‘MD’/’WD’/’XD’. має NULL допоки матч не завершиться |
| created_by_user_id | INT | NOT NULL, FK > users.id | організатор, який створив матч |
| status | ENUM('scheduled', 'ongoing', 'completed', 'cancelled') | DEFAULT 'scheduled' | статус матчу: заплановано/в процесі/завершено/скасовано |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | час коли було створено матч |

## таблиця match_participants

зв’язувальна таблиця для учасників та матчів, яка також назначає учасникам конкретну сторону майданчика

| Field | Type | Constraint | Description |
| --- | --- | --- | --- |
| id | INT | PK, AUTO_INCREMENT | унікальний ідентифікатор для учасників матчу |
| match_id | INT | NOT NULL, FK > matches.id | в якому матчі учасник бере участь |
| player_id | INT | NULL, FK > users.id | один гравець. Використовується при match_category = ‘MS’ або ‘WS’ |
| team_id | INT | NULL, FK > teams.id | команда. Використовується, якщо match_category = ‘MD’/’WD’/’XD’ |
| side | ENUM('side_a', 'side_b') | NOT NULL | сторона майданчику учасників |
| (match_id, side) | | UNIQUE | забезпечує те, що один матч має тільки одне значення side_a і одне значення side_b |

## таблиця match_sets

записує рахунок для кожного сету всередині матчу

| Field | Type | Constraint | Description |
| --- | --- | --- | --- |
| id | INT | PK, AUTO_INCREMENT | унікальний ідентифікатор сетів у матчі |
| match_id | INT | NOT NULL, FK > matches.id | до якого матчу належить сет |
| set_number | TINYINT | NOT NULL | номер сету (1/2/…) |
| side_a_score | TINYINT | NOT NULL | фінальний рахунок для гравця/команди на стороні a |
| side_b_score | TINYINT | NOT NULL | фінальний рахунок для гравця/команди на стороні b |
| winner_side | ENUM('side_a', 'side_b') | NOT NULL | сторона-переможець: side_a/side_b |
| (match_id, set_number) | | UNIQUE | запобігає створенню повторних сетів в одному матчі |

## Зв’язки

## таблиця users_roles (1:N)

`user_roles.user_id > users.id`

один користувач може мати декілька ролей (бути і гравцем і організатором водночас)

## таблиця tournaments (N:1)

`tournaments.created_by_user_id > users.id`

один організатор може створити багато турнірів

## таблиця tournament_participants (N:N між users та tournaments)

`tournament_participants.user_id > users.id`

один гравець може брати участь у багатьох турнірах (1:N)

`tournament_participants.tournament_id > tournaments.id`

один турнір може мати багато учасників (1:N)

## таблиця teams (зв’язки)

`teams.tournament_id > tournaments.id`

один турнір може мати багато команд (1:N)

`teams.player1_id > users.id`

`teams.player2_id > users.id`

один гравець може бути у багатьох командах (1:N)

## таблиця matches (зв’язки)

`matches.tournament_id > tournaments.id`

один турнір може містити багато матчів (1:N)

`matches.winner_player_id > users.id`

`matches.winner_team_id > teams.id`

один гравець/команда може бути переможцем у багатьох матчах (одиночна/парна гра)

`matches.created_by_user_id > users.id`

один організатор може створити багато матчів (1:N)

## таблиця match_participants (зв’язки)

`match_participants.match_id > matches.id`

один матч завжди має гравців на двох сторонах майданчика (по гравцю або по команді) (1:N)

`match_participants.player_id > users.id`

один гравець може брати участь у багатьох одиночних матчах (1:N)

`match_participants.team_id > teams.id`

одна команда може брати участь у багатьох парних матчах (1:N)

## таблиця match_sets (зв’язки)

`match_sets.match_id > matches.id`

один матч складається з багатьох сетів (1:N)
