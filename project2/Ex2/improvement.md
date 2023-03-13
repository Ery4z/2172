# New tables

Created new table `Match_player_info` and `Match_team_info` to remove the very high number of foreign keys in the `Match` table that all point to players.

Here is the schema for the new tables:

## Match_player_info

| Column name      | Type  | Metadata | Description                                  |
| :--------------- | :---- | :------- | :------------------------------------------- |
| `id`             | `int` | `PK`     | The id of the row                            |
| `match_api_id`   | `int` | `FK`     | The match id                                 |
| `team_api_id`    | `int` | `FK`     | The team id                                  |
| `player_api_id`  | `int` | `FK`     | The player id                                |
| `position_index` | `int` |          | The position index of the player in the team |
| `X`              | `int` |          | The X coordinate of the player in the match  |
| `Y`              | `int` |          | The Y coordinate of the player in the match  |

---

## Match_team_info

| Column name    | Type  | Metadata | Description                            |
| :------------- | :---- | :------- | :------------------------------------- |
| `id`           | `int` | `PK`     | The id of the row                      |
| `team_api_id`  | `int` | `FK`     | The team id                            |
| `match_api_id` | `int` | `FK`     | The match id                           |
| `goal`         | `int` |          | The number of goals scored by the team |
| `away`         | `int` |          | 1 if the team is away, 0 otherwise     |

---

Modified the `Match` table to remove the foreign keys to players and teams.
The following columns were removed:

-   `home_team_api_id`
-   `away_team_api_id`
-   `home_player_...`
-   `home_player_X...`
-   `home_player_Y...`
-   `away_player_...`
-   `away_player_X...`
-   `away_player_Y...`
-   `home_team_goal`
-   `away_team_goal`
