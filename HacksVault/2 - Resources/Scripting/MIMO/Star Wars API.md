1. Launch into the galaxy with a  project that calls the Star Wars API to retrieve characters from the expansive SW universe.

```python
import requests

option = "people"
url = f"https://swapi.mimo.dev/api/{option}/"

try:
  response = requests.get(url)
  response.raise_for_status()
  data = response.json()
  print(f"Successfully fetched {len(data)} entities")

except requests.HTTPError as e:
  print(f"Error fetching data: {e}")

if data:
  for entity in data:
	print(entity["name"])
else:
  print("Unable to download data")
```

2. Give power to the users in this sequel, allowing them to choose how many characters to download and explore

```python
import requests

def fetch_data(option):
  url = f"https://swapi.mimo.dev/api/{option}/"
  data = []
  try:
	response = requests.get(url)
	response.raise_for_status()
	data = response.json()
	print(f"Successfully fetched {len(data)} entities")
  except requests.HTTPError as e:
	print(f"Error fetching data: {e}")
	return None

  return data

option = input("Enter an option (e.g., 'people' or 'planets'): ").strip().lower()
data = fetch_data(option)

if data:
  for entity in data:
	print(entity["name"])
  else:
	print("Unable to download data")
```
