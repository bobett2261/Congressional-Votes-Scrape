import os
import requests
from lxml import etree
import sqlite3

def fetch_roll_call_data(roll_call_number, year=2023):
    """Fetch and parse roll call vote data from the XML URL."""
    url = f"https://clerk.house.gov/evs/{year}/roll{str(roll_call_number).zfill(3)}.xml"
    response = requests.get(url)
    if response.status_code == 200:
        return etree.XML(response.content)
    else:
        print(f"Failed to fetch data for roll call {roll_call_number}")
        return None

def parse_xml_data(xml_data):
    """Extract relevant data from the XML."""
    metadata = xml_data.find(".//vote-metadata")
    totals = xml_data.find(".//vote-totals/totals-by-vote")
    data = {
        'Majority': metadata.findtext("majority", default='N/A'),
        'Congress': metadata.findtext("congress", default='N/A'),
        'Session': metadata.findtext("session", default='N/A'),
        'Chamber': metadata.findtext("chamber", default='N/A'),
        'Roll Call Number': metadata.findtext("rollcall-num", default='N/A'),
        'Legislation Number': metadata.findtext("legis-num", default='N/A'),
        'Vote Question': metadata.findtext("vote-question", default='N/A'),
        'Vote Type': metadata.findtext("vote-type", default='N/A'),
        'Vote Result': metadata.findtext("vote-result", default='N/A'),
        'Action Date': metadata.findtext("action-date", default='N/A'),
        'Action Time': metadata.find(".//action-time").attrib.get("time-etz", 'N/A'),
        'Description': metadata.findtext("vote-desc", default='N/A'),
        'Total Yeas': totals.findtext("yea-total", default='N/A') if totals is not None else 'N/A',
        'Total Nays': totals.findtext("nay-total", default='N/A') if totals is not None else 'N/A',
        'Total Present': totals.findtext("present-total", default='N/A') if totals is not None else 'N/A',
        'Total Not Voting': totals.findtext("not-voting-total", default='N/A') if totals is not None else 'N/A'
    }
    return data

def parse_member_votes(xml_data):
    """Extract each member's vote from the XML."""
    member_votes = []
    for vote_element in xml_data.findall(".//recorded-vote"):
        legislator_element = vote_element.find('legislator')
        member_vote_data = {
            'Member Name': legislator_element.text,
            'State': legislator_element.get('state'),
            'Party': legislator_element.get('party'),
            'Vote': vote_element.findtext('vote', default='N/A')
        }
        member_votes.append(member_vote_data)
    return member_votes

def update_database_with_votes(data, member_votes, db_path):
    """Update the SQLite database with the new data including each member's vote."""
    # Ensure the directory exists
    os.makedirs(os.path.dirname(db_path), exist_ok=True)

    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()

    # Create the main table if it doesn't exist
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS roll_call_votes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        Majority TEXT,
        Congress TEXT,
        Session TEXT,
        Chamber TEXT,
        RollCallNumber TEXT,
        LegislationNumber TEXT,
        VoteQuestion TEXT,
        VoteType TEXT,
        VoteResult TEXT,
        ActionDate TEXT,
        ActionTime TEXT,
        Description TEXT,
        TotalYeas INTEGER,
        TotalNays INTEGER,
        TotalPresent INTEGER,
        TotalNotVoting INTEGER,
        MemberName TEXT,
        State TEXT,
        Party TEXT,
        Vote TEXT
    )
    ''')

    # Insert data into the main table
    for member_vote in member_votes:
        cursor.execute('''
        INSERT INTO roll_call_votes (
            Majority, Congress, Session, Chamber, RollCallNumber, LegislationNumber, VoteQuestion, 
            VoteType, VoteResult, ActionDate, ActionTime, Description, TotalYeas, TotalNays, 
            TotalPresent, TotalNotVoting, MemberName, State, Party, Vote
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        ''', (
            data['Majority'], data['Congress'], data['Session'], data['Chamber'], data['Roll Call Number'], 
            data['Legislation Number'], data['Vote Question'], data['Vote Type'], data['Vote Result'], 
            data['Action Date'], data['Action Time'], data['Description'], data['Total Yeas'], data['Total Nays'], 
            data['Total Present'], data['Total Not Voting'], member_vote['Member Name'], member_vote['State'], 
            member_vote['Party'], member_vote['Vote']
        ))

    conn.commit()
    conn.close()

def main(start_roll_call, end_roll_call, db_path, year=2023):
    """Main function to fetch, parse, and update database for a range of roll call numbers."""
    for roll_call_number in range(start_roll_call, end_roll_call + 1):
        xml_data = fetch_roll_call_data(roll_call_number, year)
        if xml_data is not None:
            data = parse_xml_data(xml_data)
            member_votes = parse_member_votes(xml_data)
            update_database_with_votes(data, member_votes, db_path)
            print(f"Successfully updated database for roll call {roll_call_number}.")
        else:
            print(f"Data for roll call {roll_call_number} could not be processed.")

if __name__ == "__main__":
    db_path = '/Users/spencer/Desktop/sql/mydatabase.db'  # Update with your actual path
    start_roll_call = 11  # Use the actual starting roll call number you wish to process
    end_roll_call = 724  # Use the actual ending roll call number you wish to process
    main(start_roll_call, end_roll_call, db_path)
