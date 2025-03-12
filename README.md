# APDA Tournament Database

## Overview
The APDA Tournament Database is designed to manage tournament details, rounds, matchups, teams, debaters, judges, and performances for American Parliamentary Debate Association (APDA) tournaments.

## Database Schema

### Tables

#### `Tournament`
Stores information about each tournament.
```sql
CREATE TABLE Tournament (
    tournament_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    date DATE NOT NULL
);
```

#### `Round`
Stores rounds within a tournament.
```sql
CREATE TABLE Round (
    round_id INT AUTO_INCREMENT PRIMARY KEY,
    tournament_id INT NOT NULL,
    round_number INT NOT NULL,
    FOREIGN KEY (tournament_id) REFERENCES Tournament(tournament_id) ON DELETE CASCADE
);
```

#### `Matchup`
Tracks matchups for each round.
```sql
CREATE TABLE Matchup (
    matchup_id INT AUTO_INCREMENT PRIMARY KEY,
    round_id INT NOT NULL,
    FOREIGN KEY (round_id) REFERENCES Round(round_id) ON DELETE CASCADE
);
```

#### `Team`
Represents debate teams.
```sql
CREATE TABLE Team (
    team_id INT AUTO_INCREMENT PRIMARY KEY,
    team_name VARCHAR(100) NOT NULL
);
```

#### `Debater`
Tracks individual debaters and their teams.
```sql
CREATE TABLE Debater (
    debater_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    experience_level ENUM('Novice', 'Varsity') NOT NULL,
    team_id INT,
    FOREIGN KEY (team_id) REFERENCES Team(team_id) ON DELETE SET NULL
);
```

#### `Role`
Defines debate roles.
```sql
CREATE TABLE Role (
    role_id INT AUTO_INCREMENT PRIMARY KEY,
    role_name ENUM('Prime Minister', 'Member of Government', 'Leader of Opposition', 'Member of Opposition') NOT NULL
);
```

#### `Judge`
Stores judge details.
```sql
CREATE TABLE Judge (
    judge_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    experience_level ENUM('Novice', 'Experienced') NOT NULL
);
```

#### `Performance`
Records performance of debaters in each round.
```sql
CREATE TABLE Performance (
    performance_id INT AUTO_INCREMENT PRIMARY KEY,
    debater_id INT NOT NULL,
    round_id INT NOT NULL,
    speaks DECIMAL(4,2) NOT NULL,
    feedback TEXT,
    FOREIGN KEY (debater_id) REFERENCES Debater(debater_id) ON DELETE CASCADE,
    FOREIGN KEY (round_id) REFERENCES Round(round_id) ON DELETE CASCADE
);
```

#### `User`
Tracks system users and their roles.
```sql
CREATE TABLE User (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    role ENUM('Organizer', 'Administrator', 'Judge') NOT NULL
);
```

#### `Tournament_User`
Associates users with tournaments.
```sql
CREATE TABLE Tournament_User (
    tournament_id INT NOT NULL,
    user_id INT NOT NULL,
    PRIMARY KEY (tournament_id, user_id),
    FOREIGN KEY (tournament_id) REFERENCES Tournament(tournament_id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES User(user_id) ON DELETE CASCADE
);
```

---

## Sample Data

### Insert Tournaments
```sql
INSERT INTO Tournament (name, date) VALUES
    ('Northeastern Invitational', '2024-12-15'),
    ('Boston Debate Open', '2025-01-20');
```

### Insert Rounds
```sql
INSERT INTO Round (tournament_id, round_number) VALUES
    (1, 1),
    (1, 2),
    (2, 1);
```

### Insert Teams & Debaters
```sql
INSERT INTO Team (team_name) VALUES ('Team Alpha'), ('Team Beta'), ('Team Gamma');

INSERT INTO Debater (name, experience_level, team_id) VALUES
    ('Alice', 'Novice', 1),
    ('Bob', 'Varsity', 1),
    ('Charlie', 'Novice', 2),
    ('Dana', 'Varsity', 3);
```

---

## Queries

### List all rounds in "Northeastern Invitational"
```sql
SELECT r.round_id, r.round_number
FROM Round r
JOIN Tournament t ON r.tournament_id = t.tournament_id
WHERE t.name = 'Northeastern Invitational';
```

### Get debater names and their teams
```sql
SELECT d.name AS Debater, t.team_name AS Team
FROM Debater d
JOIN Team t ON d.team_id = t.team_id;
```

### Show performance feedback for a given round
```sql
SELECT d.name AS Debater, p.speaks, p.feedback
FROM Performance p
JOIN Debater d ON p.debater_id = d.debater_id
WHERE p.round_id = 2;
```

---

## Stored Procedures

### Procedure to Add a New Tournament
```sql
DELIMITER $$
CREATE PROCEDURE AddTournament(IN tournamentName VARCHAR(100), IN tournamentDate DATE)
BEGIN
    INSERT INTO Tournament (name, date) VALUES (tournamentName, tournamentDate);
END$$
DELIMITER ;
```

### Procedure to Assign Judges to a Tournament
```sql
DELIMITER $$
CREATE PROCEDURE AssignJudgeToTournament(IN judgeId INT, IN tournamentId INT)
BEGIN
    INSERT INTO Tournament_User (tournament_id, user_id) VALUES (tournamentId, judgeId);
END$$
DELIMITER ;
```

---

## Triggers

### Prevent Duplicate Entries in `Tournament_User`
```sql
DELIMITER $$
CREATE TRIGGER prevent_duplicate_tournament_user
BEFORE INSERT ON Tournament_User
FOR EACH ROW
BEGIN
    IF EXISTS (
        SELECT 1 FROM Tournament_User
        WHERE tournament_id = NEW.tournament_id AND user_id = NEW.user_id
    ) THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Duplicate entry for Tournament_User.';
    END IF;
END$$
DELIMITER ;
```

---

## Views

### View Tournament Details with Number of Rounds
```sql
CREATE VIEW TournamentDetails AS
SELECT t.tournament_id, t.name AS Tournament, t.date AS Date, COUNT(r.round_id) AS NumberOfRounds
FROM Tournament t
LEFT JOIN Round r ON t.tournament_id = r.tournament_id
GROUP BY t.tournament_id;
```

### View Debater Performance Overview
```sql
CREATE VIEW DebaterPerformanceOverview AS
SELECT d.name AS Debater, t.name AS Tournament, AVG(p.speaks) AS AvgSpeaks
FROM Debater d
JOIN Performance p ON d.debater_id = p.debater_id
JOIN Round r ON p.round_id = r.round_id
JOIN Tournament t ON r.tournament_id = t.tournament_id
GROUP BY d.debater_id, t.tournament_id;
```

---

