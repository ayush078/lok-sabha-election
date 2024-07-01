---

# Election Results Data Scraping and Visualization

This report outlines the process and code used to scrape and analyze election results data from the Election Commission of India's website. The primary goal is to fetch party-wise and candidate-wise election results, format the data, and visualize key metrics.

## 1. Fetching and Parsing HTML Content

We start by defining functions to fetch and parse HTML content from given URLs.

### URLs
- *Party-wise Results:* https://results.eci.gov.in/AcResultGenJune2024/partywiseresult-S01.htm
- *Candidates-wise Results:* https://results.eci.gov.in/AcResultGenJune2024/candidateswise-S01160.htm

### Function: fetch_and_parse
python
def fetch_and_parse(url):
    response = requests.get(url)
    if response.status_code == 200:
        return BeautifulSoup(response.text, 'html.parser')
    else:
        print(f"Failed to retrieve data from {url}")
        return None


## 2. Extracting Data

### Generic Data Extraction
A function to extract text data from HTML elements with a specific class name.

### Function: extract_data
python
def extract_data(soup, class_name):
    extracted_data = []
    if soup:
        data_elements = soup.find_all('div', class_=class_name)
        for element in data_elements:
            extracted_data.append(element.text.strip())
    return extracted_data


## 3. Party-wise Data Extraction

A specific function to extract party-wise election results from the parsed HTML content.

### Function: extract_partywise_data
python
def extract_partywise_data(soup):
    party_data = []
    if soup:
        rows = soup.find_all('tr')
        for row in rows:
            cols = row.find_all('td')
            if len(cols) >= 3:
                party_name = cols[0].text.strip()
                seats_won = cols[1].text.strip()
                party_data.append((party_name, seats_won))
    return party_data


## 4. Fetch and Parse Content

Fetch and parse the content from the URLs.

python
soup_partywise = fetch_and_parse(url_partywise)
soup_candidateswise = fetch_and_parse(url_candidateswise)


## 5. Extract Data

Extract the party-wise and candidates-wise data.

python
extracted_data_partywise = extract_partywise_data(soup_partywise)
extracted_data_candidateswise = extract_data(soup_candidateswise, 'col-md-4 col-12')


## 6. Display Results

### Party-wise Results
python
print("Party-wise Results:")
for party, seats in extracted_data_partywise:
    print(f"{party}: {seats}")


### Candidates-wise Results
python
print("\nCandidates-wise Results:")
if len(extracted_data_candidateswise) == 0:
    print("YOLO")
for data in extracted_data_candidateswise:
    lines = [line for line in data.split('\n') if line.strip()]
    status = lines[0]
    margin = lines[1]
    name = lines[2] if len(lines) > 2 else ""
    party = lines[3] if len(lines) > 3 else "NA"

    print(f"status -> {status}")
    print(f"margin -> {margin}\n")
    print(f"name -> {name}")
    print(f"party -> {party}\n")


## 7. Extracting Additional Data

Fetching and parsing additional data from another URL and organizing it in a dictionary format.

### Function to Fetch and Parse Additional Data
python
url = "https://results.eci.gov.in/"
fetch_results = fetch_and_parse(url)
data = extract_data(fetch_results, "row justify-content-center")
data_str = data[0]


### Formatting the Data
python
lines = [line.strip() for line in data_str.split('\n') if line.strip()]
formatted_data = {}
current_key = None

for line in lines:
    if line.isdigit():
        if current_key:
            formatted_data[current_key] = int(line)
    else:
        if 'Constituencies' not in line:
            current_key = line

for key, value in formatted_data.items():
    print(f"{key} -> {value}")


## 8. Visualizing Data

Fetching detailed constituency-level data and visualizing the margin of victory.

### URL
- *State-wise Results:* https://results.eci.gov.in/AcResultGen2ndJune2024/statewiseS021.htm

### Extract Data and Convert to DataFrame
python
fetch_results = fetch_and_parse(url)
data = extract_data(fetch_results, "table table-striped table-bordered")
df = pd.DataFrame(data)


### Plotting the Data
python
plt.figure(figsize=(15, 10))
plt.bar(df['Constituency'], range(len(df['Constituency'])), color='skyblue')
plt.xlabel('Constituency')
plt.ylabel('Margin of Victory')
plt.title('Margin of Victory for Each Constituency')
plt.xticks(rotation=90)
plt.show()
---
