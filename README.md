# ESPN-NBA

Functions to retrieve data from ESPN NBA.

    This is a library aimed at getting data from ESPN,
    regarding NBA matches. A variety of data is available,
    as a number of available ways to acquire them is provided.

    A more detailed list follows below:

    Available data:
        - quarter scores
        - play by play
        - box score
        - team totals

    Available ways:
        - single matches, requires game_id
        - single days, requires datetime object
        - range of days, requires dates as strings
        - update previous saved data, requires csv file

    Misc:
        - save to csv, xls, xlsx
        - url handling with mechanize or requests

    Date format used "%d/%m/%Y".
    Can be changed in individual functions.

    Data structure is ideal for use with Pandas.

    >> Examples:

        Single match:
            target_date = datetime.date(2015, 1, 10)
            espn_scoreboard = format_scoreboard_url(target_date)
            game_ids = scrape_box_score_links(espn_scoreboard)
            game_id example: "gameId=400578860"
            data = parse_box_score(game_ids[0])

        Single day:
            target_date = datetime.date(2015, 1, 15)
            data = get_data("score", target_date)
            save_to_file(data, "score_15_1_2015.csv")

        Range of days:
            data = get_data_by_daterange("box score", "1/1/2015", "31/1/2015")

        Update:
            data = get_update("score", "scores.csv")
