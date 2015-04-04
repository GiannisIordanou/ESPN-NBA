# -*- coding: utf-8 -*-

import re
import os
from functools import partial
import requests
import mechanize
import cookielib
from urlparse import urlparse
from bs4 import BeautifulSoup
import datetime
from dateutil.parser import parse
import pandas as pd

__author__ = "Iordanou Giannis"
__copyright__ = ""
__credits__ = ""
__license__ = "GPL"
__version__ = "0.1"
__maintainer__ = "Iordanou Giannis"
__email__ = "iordanougiannis@hotmail.gr"
__status__ = "Developing"

# Functions

# Read url Functions


def read_url_with_mechanize(url):
    """
    Open and read url

    Parameters
    ----------
    url : string
          Url to open and read

    Returns
    -------
    html : string
           The source of the url
    """

    # Browser
    br = mechanize.Browser()

    # Cookie Jar
    cj = cookielib.LWPCookieJar()
    br.set_cookiejar(cj)

    # Browser options
    br.set_handle_equiv(True)
    #br.set_handle_gzip(True)
    br.set_handle_redirect(True)
    br.set_handle_referer(True)
    br.set_handle_robots(False)

    # Follows refresh 0 but not hangs on refresh > 0
    br.set_handle_refresh(mechanize._http.HTTPRefreshProcessor(), max_time=1)

    # User-Agent (this is cheating, ok?)
    headers = [('User-agent', 'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.1) \
                Gecko/2008071615 Fedora/3.0.1-1.fc9 Firefox/3.0.1')]
    br.addheaders = headers

    r = br.open(url)
    html = r.read()

    return html


def read_url_with_requests(url):
    """
    Open and read url

    Parameters
    ----------
    url : string
          Url to open and read

    Returns
    -------
    html : string
           The source of the url
    """
    headers = {'User-agent': 'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.1) \
         Gecko/2008071615 Fedora/3.0.1-1.fc9 Firefox/3.0.1'}
    r = requests.get(url, headers=headers)
    html = r.text
    return html


def read_url(url, read_with, retries=5):
    """
    Opens and reads url with selected function
    retrying as many times as retries indicates if error

    Parameters
    ----------
    url : string
          Url to open and read
    read_with : string
                Module to use, mechanize or requests
    retries : int - default 5
              Number of tries if error

    Returns
    -------
    html : string
           The source of the url

    """
    readers = {"mechanize": read_url_with_mechanize, "requests": read_url_with_requests}
    if read_with in readers:
        errors = 0
        html = None
        while errors <= retries:
            try:
                html = readers[read_with](url)
            except Exception, e:
                print "Read URL Error:", e
                print url
                html = None
                errors += 1
            if html:
                break
    else:
        html = None
        print "Choose mechanize or requests as read_with."
    return html


# Save Function

def save_to_file(data, save_filename, columns_order=None): # TODO: Fix docstring
    """
    Save data to file

    Parameters
    ----------
    data: 
          data to save
    
    save_filename: string
                   
    
    columns_order: list
                   1-D list of column names

    Returns
    -------
    a string of the source of the url

    """

    EXPORTERS = {".xls": pd.DataFrame.to_excel,
                 ".xlsx": pd.DataFrame.to_excel,
                 ".csv": partial(pd.DataFrame.to_csv, index=False)}
    try:
        df = pd.DataFrame(data)
        if columns_order:  # Rearrange columns if needed
            df = df[columns_order]

        _, ext = os.path.splitext(save_filename)
        if ext in EXPORTERS: # Save to file
            EXPORTERS[ext](df, save_filename)
            return True
        else:
            raise Exception("Not a supported filetype: {}".format(ext))
    except Exception, e:
        print "Save_to_file error:", e


def format_scoreboard_url(day, league='nba'):
    """
    Format ESPN scoreboard link

    Parameters
    ----------
    day : datetime object or string
          The day of the scoreboard # TODO: Fix parameter

    Returns
    -------
    scoreboard_link : string
                      The scoreboard link
                      for the specified day

    Examples
    --------
    >>scoreboard_link = format_scoreboard_url("20150101")

    >> day = datetime.date(2015,1,1)
    >> scoreboard_link = format_scoreboard_url(day)

    """
    league = league.lower()

    if isinstance(day, datetime.date):
        date_param = day.strftime('%Y%m%d')
    else:
        date_param = day

    scoreboard_link = ''.join(['http://scores.espn.go.com/', league, '/scoreboard?date=', date_param])
    return scoreboard_link


# Scrape Functions


def scrape_links(espn_scoreboard):
    """
    Scrape ESPN's scoreboard for Play-By-Play links.

    Parameters
    ----------
    espn_scoreboard : string
                      The url to scrape

    Returns
    -------
    queries : list
              1-D list of strings, of the following format:
              "gameId=400578860"

    """
    html = read_url(espn_scoreboard, "mechanize")
    if html:
        soup = BeautifulSoup(html, ['fast', 'lxml'])
        div = soup.find('div', {'class': 'span-4'})
        links = (a['href'] for a in div.findAll('a') if re.match('Play.*',
                 a.contents[0]))
        queries = [urlparse(link).query for link in links]
        print 1
    else:
        queries = None
    return queries


def scrape_box_score_links(espn_scoreboard):
    """
    Scrape ESPN's scoreboard for Box Score links.

    Parameters
    ----------
    espn_scoreboard : string
                      The url to scrape.

    Returns
    -------
    queries : list
              1-D list of strings, of the following format:
              "gameId=400578860".

    """
    html = read_url(espn_scoreboard, "mechanize")
    if html:
        links = list(set(re.findall('href="(http://espn.go.com/nba/boxscore\?gameId=[0-9]{1,})"', html)))
        queries = [urlparse(link).query for link in links]
    else:
        queries = None
    return queries


def scrape_game_ids(espn_scoreboard): # TODO: Fix the type of gameids
    """
    Scrape ESPN's scoreboard for game ids.

    Parameters
    ----------
    espn_scoreboard : string
                      The url to scrape.

    Returns
    -------
    queries : list
              1-D list of strings, of the following format:
              "400578860".

    """
    html = read_url(espn_scoreboard, "mechanize")
    if html:
        game_ids = sorted(list(set(re.findall('href="/nba/.*?\?gameId=([0-9]{1,})">', html))))
    else:
        game_ids = None
    return game_ids


# Parse Functions


def parse_score(game_id, league='nba'):  # TODO: Fix docstring
    """"
    Parse a game's box score page for scores.

    Paramaters
    ----------
    game_id : string

    Returns
    -------
    match_score : dictionary
                  1-D dictionary containing details about
                  a  matches points

    """
    columns_names = ["Season", "League", "Date", "Time", "Series_status", "Status", "OT", "HomeTeam", "AwayTeam",
                     "Q1HP", "Q2HP", "Q3HP", "Q4HP", "QOT1HP", "QOT2HP", "QOT3HP", "QOT4HP", "QOT5HP", "QOT6HP",
                     "Q1AP", "Q2AP", "Q3AP", "Q4AP", "QOT1AP", "QOT2AP", "QOT3AP", "QOT4AP", "QOT5AP", "QOT6AP",
                     "HP", "AP", "FTR", "TP", "Stadium", "City", "State", "Official_1", "Official_2", "Official_3",
                     "Attendance", "ToG", "Game_id"]
    #print "Game_id:", game_id

    #start = time.time()
    league = league.lower()
    espn_url = 'http://scores.espn.go.com/' + league + '/boxscore?' + game_id
    html = read_url(espn_url, "mechanize")

    #stop = time.time() - start
    #print "Read url in:", round(stop, 2), ".s"
    try:
        game_id = int(game_id.split("=")[1])
    except Exception, e:
        game_id = game_id

    if html:
        soup = BeautifulSoup(html, ['fast', 'lxml'])

        # Status
        try:
            status = soup.find("p", {"class": "game-state"})
            status = status.text
            ot = "OT" if "OT" in status else ""
            status = status.split("/")[0].strip()
        except Exception, e:
            print "Status Error:", e
            #print espn
            status, ot = ""

        try:
            series_status = re.findall("(.*?) \(", soup.find("div", {"class": "series-status"}).text)[0]
        except Exception, e:
            print "Series Status Error:", e
            #print espn
            series_status = ""

        try:
            away_team, home_team = map(lambda x: x.strip(), re.findall(".*? -", soup.title.string.replace("\n", "").replace("\r", ""))[0].replace(" -", "").split("vs."))
        except Exception, e:
            print "Team Error:", e
            #print espn
            away_team, home_team = "", ""
            print soup.title.string

        if "Stars" in home_team:
            series_status = "All Stars"

        try:
            match_date, location = map(lambda x: x.text, soup.find("div", {"class": re.compile("game.*?time-location")}).findAll("p"))
            dt = parse(match_date)
            match_date = dt.date().strftime("%d/%m/%Y")
            match_time = dt.time().strftime("%H:%M")
            season = dt.year + 1 if 9 <= dt.month <= 12 else dt.year
        except Exception, e:
            print "Match date Error:", e
            #print espn
            match_date, location, match_time, season = "", "", "", ""

        try:
            if location:
                if len(location.split(",")) == 3:
                    stadium, city, state = map(lambda x: x.strip(), location.split(","))
                elif len(location.split(",")) == 2:
                    stadium, city = map(lambda x: x.strip(), location.split(","))
                    state = ""
                elif len(location.split(",")) == 1:
                    stadium = location
                    city, state = "", ""
                else:
                    print "Stadium", location
                    #print espn
                    stadium, city, state = "", "", ""
            else:
                stadium, city, state = "", "", ""
        except Exception, e:
            print "Stadium date Error:", e
            #print espn
            stadium, city, state = "", "", ""
            #stadium, city, state = map(lambda x: x.strip(), location.split(",")) + [""]

        try:
            quarters = map(lambda x: x.text, soup.findAll("td", {"class":"period", "style":"text-align:center"}))
        except Exception, e:
            print "Quarters Error:", e
            #print espn
            quarters = ""

        try:
            quarter_points = map(float, [i.text for i in soup.findAll("td",{"class": "", "colspan": "", "style":"text-align:center"})])
            #quarter_points = [] Error when float
        ##print espn
        #print quarter_points
            if not quarters:
                quarters = len(quarter_points) / 2
            home_quarters_points = quarter_points[-len(quarters):] + (10 - len(quarters)) * [""]
            away_quarters_points = quarter_points[:len(quarters)] + (10 - len(quarters)) * [""]
        except Exception, e:
            print "Quarters points Error:", e
            #print espn
            home_quarters_points = ["", "", "", "", "", "", "", "", "", ""]
            away_quarters_points = ["", "", "", "", "", "", "", "", "", ""]

        try:
            home_points, away_points = map(lambda x: float(x.text), soup.findAll("td", {"class":"ts", "style":"text-align:center"}))[::-1]
        except Exception, e:
            print "Team points Error:", e
            #print espn
            try:
                home_points, away_points = map(lambda x: float(x.span.text), soup.findAll("h3"))[::-1]
            except Exception, e:
                print "Team points Error 2:", e
                #print espn
                home_points, away_points = "", ""

        try:
            total_points = sum([home_points, away_points])
        except Exception, e:
            print "Total points Error:", e
            #print espn
            total_points = ""

        try:
            ftr_dict = {-1: "A", 0: "D", 1: "H"}
            if home_points and away_points:
                ftr = ftr_dict[cmp(home_points, away_points)]
            else:
                ftr = ""
        except Exception, e:
            print "FTR Error:", e
            #print espn
            ftr = ""

        try:
            officials = map(lambda x: x.strip(), re.findall("Officials:</strong> (.*?)<br>", html)[0].split(","))
            officials.extend((3 - len(officials)) * [""])
        except Exception, e:
            print "Officials Error:", e
            #print espn
            officials = ["", "", ""]
        try:
            attendance = float(re.findall("Attendance:</strong> (.*?)<br>", html)[0].replace(",", ""))
        except Exception, e:
            print "Attendance Error:", e
            #print espn
            attendance = ""

        try:
            hours, minutes = map(float, re.findall("Time of Game:</strong> (.*?)<br>", html)[0].split(":"))
            time_of_game = hours * 60. + minutes
        except Exception, e:
            print "Time of game Error:", e
            #print espn
            time_of_game = ""
        match_score = [season, league.upper(), match_date, match_time, series_status, status, ot, home_team, away_team] + home_quarters_points + away_quarters_points + [home_points, away_points] + [ftr, total_points, stadium, city, state] + officials + [attendance, time_of_game, game_id]

        match_score = dict(zip(columns_names, match_score))

        #stop1 = time.time() - start
        #print "Scraping in:", round(stop1, 2), ".s"
        return match_score
    else:
        match_score = (len(columns_names) - 1) * [""]
        match_score = dict(zip(columns_names, match_score))
        match_score["Game_id"] = game_id
        #match_score = [match_score]
        return match_score  # TODO: Fix when can't read url


def parse_box_score(game_id, league="nba"):
    league = league.lower()
    espn_url = 'http://scores.espn.go.com/' + league + '/boxscore?' + game_id
    html = read_url(espn_url, "mechanize")

    try:
        game_id = int(game_id.split("=")[1])
    except Exception, e:
        game_id = game_id

    if html:
        league = league.upper()
        html_rows = html.split("\n")
        soup = BeautifulSoup(html, ['fast', 'lxml'])

        starters_ind = [index for index, i in enumerate(html_rows) if "STARTERS" in i]
        bench_ind = [index for index, i in enumerate(html_rows) if "BENCH" in i]

        stats_cat = [filter(lambda x: x, i.text.split("\n")) for i in soup.findAll("tr", {"class": "", "align": "right"})][0][1:]

        players_dicts = []

        try:
            away_team, home_team = map(lambda x: x.strip(), re.findall(".*? -", soup.title.string.replace("\n", "").replace("\r", ""))[0].replace(" -", "").split("vs."))
        except Exception, e:
            print "Team Error:", e
            ##print espn
            away_team, home_team = "", ""

        try:
            match_date, location = map(lambda x: x.text, soup.find("div", {"class": re.compile("game.*?time-location")}).findAll("p"))
            dt = parse(match_date)
            match_date = dt.date().strftime("%d/%m/%Y")
            match_time = dt.time().strftime("%H:%M")
            season = dt.year + 1 if 9 <= dt.month <= 12 else dt.year
        except Exception, e:
            print "Match date Error:", e
            ##print espn
            match_date, location, match_time, season = "", "", "", ""

        #print ">> Game_id:", game_id, type(game_id)

        for index, row in enumerate(html_rows):
            if '<td style="text-align:left" nowrap><a href=' in row:
                player_name = re.findall('">(.*?)<', row.split("/a>")[0])[0]
                player_index = index
                player_pos = re.findall(" (\w{1,})", row.split("/a>")[1])[0]
                player_stats_values = re.findall("<td>(.*?)</td>", "".join(row.split("/a>")[1:]).replace("td align=right>", "td>"))
                player_stats = []
                for player_stat in player_stats_values:
                    #print player_stat,
                    if player_stat:
                        try:
                            player_stats.append(float(player_stat))
                            #print "float"
                        except Exception, e:
                            #print e
                            player_stats.append(player_stat)
                    else:
                        player_stats.append(player_stat)


                player_dict = dict(zip(stats_cat, player_stats))
                player_dict["Season"] = season
                player_dict["Date"] = match_date
                player_dict["HomeTeam"] = home_team
                player_dict["AwayTeam"] = away_team
                player_dict["Name"] = player_name
                player_dict["Position"] = player_pos

                if starters_ind[0] < player_index < bench_ind[0]:
                    status = "Starter"
                    team = away_team
                elif bench_ind[0] < player_index < starters_ind[1]:
                    status = "Bench"
                    team = away_team
                #else:
                    #status, team = "", "100"

                if starters_ind[1] < player_index < bench_ind[1]:
                    status = "Starter"
                    team = home_team
                elif bench_ind[1] < player_index:
                    status = "Bench"
                    team = home_team
                #else:
                    #status, team = "", "200"
                    #print starters_ind, player_index, bench_ind

                player_dict["Status"] = status
                player_dict["Team"] = team

                player_dict["Game_id"] = game_id
                for stat in stats_cat:
                    if stat not in player_dict.keys():
                        player_dict[stat] = ""

                for key in player_dict.keys():
                    if "M-A" in key:
                        value = player_dict[key]
                        new_key = key.replace("M-A", "")
                        player_dict[new_key + "M"] = float(value.split("-")[0]) if "-" in value else value

                        player_dict[new_key + "A"] = float(value.split("-")[1]) if "-" in value else value
                        player_dict[new_key + "P"] = round(player_dict[new_key + "M"] / player_dict[new_key + "A"], 2) if player_dict[new_key + "A"] != 0 and player_dict[new_key + "A"] != "" else 0.
                        player_dict.pop(key, None)

                players_dicts.append(player_dict)

        return players_dicts
    else:
        return {"": ""}# TODO: Fix when can't read url


def parse_play_by_play(game_id, league="nba"):
    #espn_link = "http://scores.espn.go.com/nba/playbyplay?gameId=" + str(game_id) + "&period=0"
    espn_link = "http://scores.espn.go.com/" + league + "/playbyplay?" + game_id + "&period=0"
    print espn_link
    html = read_url(espn_link, "mechanize")

    try:
        game_id = int(game_id.split("=")[1])
    except Exception, e:
        game_id = game_id

    if html:

        soup = BeautifulSoup(html)

        # Standard
        try:
            away_team, home_team = map(lambda x: x.strip(), re.findall(".*? -", soup.title.string.replace("\n", "").replace("\r", ""))[0].replace(" -", "").split("vs."))
        except Exception, e:
            print "Team Error:", e
            away_team, home_team = "", ""

        print home_team, away_team

        try:
            match_date, location = map(lambda x: x.text, soup.find("div", {"class": re.compile("game.*?time-location")}).findAll("p"))
            dt = parse(match_date)
            match_date = dt.date().strftime("%d/%m/%Y")
            match_time = dt.time().strftime("%H:%M")
            season = dt.year + 1 if 9 <= dt.month <= 12 else dt.year
        except Exception, e:
            print "Match date Error:", e
            #print espn
            match_date, location, match_time, season = "", "", "", ""

        print match_date, location, match_time, season

        play_by_play = []
        rows = re.findall('<tr class="(odd|even)">(.*?)</tr>', html)
        if rows:

            quarter = 1
            previous_time = 0
            end_of_quarter = False

            for row in rows:
                play_row = row[1].replace("&nbsp;", "")
                play_dict = {"Season": season, "Date": match_date, "Match Time": match_time,
                             "HomeTeam": home_team, "AwayTeam": away_team, "Home Play": "",
                             "Away Play": "", "Official Play": "", "Game_id": game_id}


                row_items = re.findall("<td.*?</td>", play_row)
                items = map(lambda x: re.sub("<.*?>", "", re.findall('>(.*?)</td', x)[0]), row_items)

                if len(items) == 4:
                    play_dict["Time"], play_dict["Away Play"], play_dict["Score"], play_dict["Home Play"] = items
                elif len(items) == 2:
                    play_dict["Time"], play_dict["Official Play"] = items
                    try:
                        play_dict["Score"] = play_by_play[-1]["Score"]
                    except Exception, e:
                        play_dict["Score"] = "0-0"

                play_dict["HP"], play_dict["AP"] = map(int, play_dict["Score"].split("-")) if play_dict["Score"] else ["", ""]
                play_dict["TP"] = sum([play_dict["HP"], play_dict["AP"]]) if play_dict["HP"] != "" and play_dict["AP"] != "" else ""
                play_dict["Q"] = quarter

                if len(row_items) > 1:
                    # Get overall time
                    minutes, seconds = map(int, play_dict["Time"].split(":")) if play_dict["Time"] else ["", ""]
                    if minutes is 0 and not end_of_quarter:
                        end_of_quarter = True
                    elif end_of_quarter and minutes > 1:
                        quarter += 1
                        end_of_quarter = False

                    num_quarters, regulation_time, regular_quarter = 4, 48, 12
                    if quarter > num_quarters:
                        quarter_length = 5
                        overtimes = quarter - num_quarters
                        previous_time = datetime.timedelta(minutes=(regulation_time + 5 * (overtimes - 1)))
                    else:
                        quarter_length = regular_quarter
                        previous_time = datetime.timedelta(minutes=(quarter_length * (quarter - 1)))
                    mins = datetime.timedelta(minutes=quarter_length) - datetime.timedelta(minutes=minutes, seconds=seconds)
                    overall_time = str(mins + previous_time)

                    play_dict["OVTT"] = overall_time
                else:
                    play_dict["OVTT"] = ""

                play_by_play.append(play_dict)
            return play_by_play

        else:
            return None

    else:
        return None  # TODO: Fix docstring


def parse_totals(game_id, league="nba"):  # TODO: Fix docstring

    league = league.lower()
    espn_url = 'http://scores.espn.go.com/' + league + '/boxscore?' + game_id
    html = read_url(espn_url, "mechanize")

    try:
        game_id = int(game_id.split("=")[1])
    except Exception, e:
        game_id = game_id

    teams_totals_dict = {}
    teams_totals_dict["Game_id"] = game_id

    if html:
        soup = BeautifulSoup(html)
        try:
            away_team, home_team = map(lambda x: x.strip(), re.findall(".*? -", soup.title.string.replace("\n", "").replace("\r", ""))[0].replace(" -", "").split("vs."))
        except Exception, e:
            print "Team Error:", e
            ##print espn
            away_team, home_team = "", ""
        teams_totals_dict["HomeTeam"] = home_team
        teams_totals_dict["AwayTeam"] = away_team

        try:
            match_date, location = map(lambda x: x.text, soup.find("div", {"class": re.compile("game.*?time-location")}).findAll("p"))
            dt = parse(match_date)
            match_date = dt.date().strftime("%d/%m/%Y")
            match_time = dt.time().strftime("%H:%M")
            season = dt.year + 1 if 9 <= dt.month <= 12 else dt.year
        except Exception, e:
            print "Match date Error:", e
            ##print espn
            match_date, location, match_time, season = "", "", "", ""

        teams_totals_dict["Season"] = season
        teams_totals_dict["Date"] = match_date
        teams_totals_dict["Time"] = match_time

        total_stats_matches = soup.findAll("tr", {"class": "", "align": "right"})
        if total_stats_matches:
            total_stats_cat = [filter(lambda x: x and u"\xa0" not in x, i.text.split("\n")) for i in total_stats_matches][2][1:]
            teams_totals_matches = soup.findAll("tr", {"class": "even", "align": "right", "valign": ""})
            if teams_totals_matches:
                home_team_totals = [ii.text for ii in teams_totals_matches[1].findAll("strong")]
                home_team_totals_dict = dict(zip(total_stats_cat, home_team_totals))
                away_team_totals = [ii.text for ii in teams_totals_matches[0].findAll("strong")]
                away_team_totals_dict = dict(zip(total_stats_cat, away_team_totals))
                for index, team_totals_dict in enumerate([home_team_totals_dict, away_team_totals_dict]):
                    for key in team_totals_dict.keys():
                        value = team_totals_dict[key]
                        team_status = "_" + ["H", "A"][index]
                        if "M-A" in key:
                            new_key = key.replace("M-A", "")
                            team_totals_dict[new_key + "M" + team_status] = float(value.split("-")[0]) if "-" in value else value

                            team_totals_dict[new_key + "A" + team_status] = float(value.split("-")[1]) if "-" in value else value
                            team_totals_dict[new_key + "P" + team_status] = round(team_totals_dict[new_key + "M" + team_status] / team_totals_dict[new_key + "A" + team_status], 3) if team_totals_dict[new_key + "A" + team_status] != 0 and team_totals_dict[new_key + "A" + team_status] != "" else 0.
                        else:
                            try:
                                team_totals_dict[key + team_status] = float(value)
                            except:
                                team_totals_dict[key + team_status] = value
                        team_totals_dict.pop(key, None)

                teams_totals_dict.update(dict(home_team_totals_dict.items() + away_team_totals_dict.items()))
                return teams_totals_dict
            else:
                return None
        else:
            return None
    else:
        return None

# Get functions


def get_data(type_of_data, day, league="nba"):  # TODO: Fix parameters
    """
    Gets data depending on what type_of_data variable
    for one date.

    Parameters
    ----------
    type_of_data : string
                   Choose between score, box score,
                   play_by_play and totals.
    day : datetime object or string
          The day of the scoreboard # TODO: Fix parameter
    league: string - default "nba"
            The league.

    Returns
    -------
    games : list
            List of lists of the data specified.

    """
    scrapers = {"score": scrape_box_score_links, "box score": scrape_box_score_links,
                "play by play": scrape_links, "totals": scrape_box_score_links}
    parsers = {"score": parse_score, "box score": parse_box_score,
               "play by play": parse_play_by_play, "totals": parse_totals}

    espn_scoreboard = format_scoreboard_url(day, league=league)
    all_games = scrapers[type_of_data](espn_scoreboard)
    games = []
    for game in all_games:
        game_data = parsers[type_of_data](game, league=league)
        if isinstance(game_data, list):
            games.extend(parsers[type_of_data](game, league=league))
        elif isinstance(game_data, dict):
            games.extend([parsers[type_of_data](game, league=league)])

    #games = sum([parsers[type_of_data](game, league=league) for game in all_games], [])
    return games


# Get by daterange Functions


def get_data_by_daterange(type_of_data, start_date, end_date, date_format="%d/%m/%Y"):  # TODO: Fix docstring
    """
    Gets data depending on what type_of_data variable
    for a range of dates

    Parameters
    ----------
    type_of_data : string
                   Choose between score, box score,
                   play_by_play and totals.
    start_date : string
                 The first day of the date range. # TODO: Fix parameter
    end_date : string
               The last day of the date range. # TODO: Fix parameter
    date_format : string
                  The format of the date


    Returns
    -------
    data : list
           List of lists of the data specified.

    Examples
    --------
    >> data = get_data_by_daterange("score", "1/1/2015", "15/1/2015")

    """
    start_date, end_date = daterange_check(start_date, end_date, date_format)
    if start_date and end_date:
        data = sum([get_data(type_of_data, day) for day in daterange(start_date, end_date)], [])
    else:
        data = None
    return data


# Date Functions


def daterange(start, end):  # TODO: Fix docstring
    """Generator for days between two specific days."""
    for n in range((end - start).days):
        yield start + datetime.timedelta(n)


def format_date(date_string, date_format="%d/%m/%Y"):  # TODO: Fix docstring
    """
    Convert string date to proper datetime object.

    Parameters
    ----------
    date_string: datetime object or string
    date_format : string
                  The format of the date

    Returns
    -------
    formated_date : datetime.datetime object

    """
    if not isinstance(date_string, datetime.datetime) or not isinstance(date_string, datetime.date):
        try:
            formated_date = datetime.datetime.strptime(date_string, date_format)
        except Exception, e:
            print "Format data error:", e
            formated_date = None
    else:
        formated_date = date_string
    return formated_date


def daterange_check(start_date, end_date, date_format="%d/%m/%Y"):  # TODO: Fix docstring
    # if not start_date:
    #     start_date = "1/10/2001"
    # if not end_date:
    #     end_date = datetime.datetime.today().strftime(date_format)

    try:
        start_date = format_date(start_date, date_format)
        end_date = format_date(end_date, date_format)
    except Exception, e:
        print "daterange_check error:", e
        start_date, end_date = None, None

    return start_date, end_date


# Update Functions


def get_update(type_of_data, update_filename, date_format="%d/%m/%Y"):  # TODO: Fix docstring
    """ Returns all data, previous and new, based on a csv file.

    Parameters
    ----------
    type_of_data : string
                   Choose between score, box score,
                   play_by_play and totals.
    update_filename: string
                     The path of a csv file.
    date_format:


    Returns
    -------
    all_data_dict : dictionary

    Examples
    --------
    >> data = get_update("score", "scores.csv")

    """
    print "Updating file:", update_filename
    file_exist = os.path.isfile(update_filename)
    if file_exist:
        # pandas
        previous_data = pd.read_csv(update_filename)
        previous_data.fillna("", inplace=True)
        start_date = format_date(previous_data[-1:]["Date"].values[0], date_format)
    else:
        start_date = None
    print "Last date:", start_date
    end_date = datetime.datetime.today().strftime(date_format)
    print "End date:", end_date

    if start_date and end_date:
        update = get_data_by_daterange(type_of_data, start_date, end_date, date_format)
    else:
        update = None

    if update:
        update_data = pd.DataFrame(update)
        all_data = pd.concat([previous_data, update_data], ignore_index=True)
        all_data.drop_duplicates(inplace=True)
        all_data.reset_index(drop=True, inplace=True)
        all_data_dict = all_data.to_dict()
    else:
        all_data_dict = None

    return all_data_dict
