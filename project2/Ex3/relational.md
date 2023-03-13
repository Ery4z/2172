# Exercice 3.

The following document the general table structure and justification about the relational schema of the database.

---

## Table structure

### Table: `VehicleClass`

| Column   | Type           | Description |
| -------- | -------------- | ----------- |
| `vcid`   | `int`          | PK          |
| `vcname` | `varchar(255)` | AK NOT NULL |
| `length` | `int`          | NOT NULL    |
| `width`  | `int`          | NOT NULL    |
| `height` | `int`          | NOT NULL    |

```sql

```

---

### Table: `Customer`

| Column  | Type           | Description                            |
| ------- | -------------- | -------------------------------------- |
| `cid`   | `int`          | PK                                     |
| `first` | `varchar(255)` | NOT NULL AK when associated with last  |
| `last`  | `varchar(255)` | NOT NULL AK when associated with first |
| `vcid`  | `int`          | FK NULLABLE                            |

```sql

```

---

### Table: `Vehicle`

In this table the attribute `is_car` allow to distinguish between a car and a bike. A car is supposed to have a non null `plate_num` but it is not possible to verify this constraint in the database. The `plate_num` is supposed to be unique. This choice has been made to ensure a vehicle is either a car or a bike but not both.

The `vid` is used as a primary key and the `num` is used as a unique key when used with the `vcid` to ensure that a vehicle is unique in a class. This choice has been made to refer to a vehicle in the table `Reservation` with the `vid` and prevent to use `num` and `vcid`.

Tuple of this table are weak entities. If the tuple for `VehicleClass` referenced by `vcid` is deleted then the tuple for `Vehicle` is deleted too. This is the same for the tuple for `Station` referenced by `sid`.

| Column            | Type           | Description                       |
| ----------------- | -------------- | --------------------------------- |
| `vid`             | `int`          | PK                                |
| `num`             | `int`          | NOT NULL AK when used with vcid   |
| `last_check_date` | `date`         | NOT NULL                          |
| `is_car`          | `boolean`      | NOT NULL                          |
| `plate_num`       | `varchar(255)` | NULLABLE                          |
| `vcid`            | `int`          | FK NOT NULL AK when used with num |
| `sid`             | `int`          | FK NOT NULL                       |

```sql

```

---

### Table: `Station`

In this table we added the attribute `sid` to be able to use a foreign key in the table `Vehicle` to refer to a station.
The use of name should have been a possibility but it is if the name of a station is modified then the foreign key will not be valid anymore.

| Column     | Type           | Description                               |
| ---------- | -------------- | ----------------------------------------- |
| `sid`      | `int`          | PK                                        |
| `name`     | `varchar(255)` | AK                                        |
| `street`   | `varchar(255)` | AK when associated with city & postcode   |
| `city`     | `varchar(255)` | AK when associated with street & postcode |
| `postcode` | `varchar(255)` | AK when associated with city & street     |

```sql

```

---

### Table: `Reservation`

The `rid` has been adde to be able to use a foreign key in the table `FinishedReservation` to refer to a reservation.

The tuple of this table are weak entities. If the tuple for `Vehicle` referenced by `vid` is deleted then the tuple for `Reservation` is deleted too. This is the same for the tuple for `Customer` referenced by `cid`.

| Column          | Type       | Description |
| --------------- | ---------- | ----------- |
| `rid`           | `int`      | PK          |
| `cid`           | `int`      | FK NOT NULL |
| `startDateTime` | `datetime` | NOT NULL    |
| `endDateTime`   | `datetime` | NOT NULL    |
| `vid`           | `int`      | FK NOT NULL |

```sql

```

---

### Table: `FinishedReservation`

| Column     | Type  | Description |
| ---------- | ----- | ----------- |
| `rid`      | `int` | PK          |
| `distance` | `int` | NOT NULL    |
| `cost`     | `int` | NOT NULL    |

```sql

```

---

## Justification

The choice of creating another table for FinishedReservation has been made reagrding the fact that it is asked to minimize the number of nullable attribute in the database. If the cost of adding a new table is not too high, it is better to create a new table than to add nullable attributes in an existing table. But in the other case it is possible to add `distance` and `cost` as nullable attributes in the table `Reservation`. This would not ensure that the `distance` and `cost` are not missing when one is not. But it would allow to have a single table for the reservations.
