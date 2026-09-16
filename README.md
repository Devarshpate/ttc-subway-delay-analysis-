# TTC Subway Delay Analysis
# Data source: City of Toronto Open Data Portal
# https://open.toronto.ca/dataset/ttc-subway-delay-data/

import requests
import pandas as pd
import matplotlib.pyplot as plt

# ------------------------------------------------------------
# 1. Load the data
# ------------------------------------------------------------
base_url = "https://ckan0.cf.opendata.inter.prod-toronto.ca"
package_url = base_url + "/api/3/action/package_show"
params = {"id": "ttc-subway-delay-data"}

package = requests.get(package_url, params=params).json()

resources = package["result"]["resources"]
csv_resources = [r for r in resources if r["format"].upper() == "CSV"]
target = csv_resources[0] if csv_resources else resources[0]
print("Using file:", target["name"])

if target.get("datastore_active"):
    data_url = base_url + "/api/3/action/datastore_search"
    resp = requests.get(data_url, params={"resource_id": target["id"], "limit": 32000}).json()
    df = pd.DataFrame(resp["result"]["records"])
else:
    df = pd.read_csv(target["url"])

print(df.shape)
print(df.head())

# ------------------------------------------------------------
# 2. Question 1: Which stations have the most delay incidents?
# ------------------------------------------------------------
delays_by_station = df.groupby("Station").size().sort_values(ascending=False).head(10)
print("\nTop 10 stations by number of delay incidents:")
print(delays_by_station)

delays_by_station.plot(kind="bar", figsize=(10, 5), color="darkorange")
plt.title("Top 10 Stations by Number of Delay Incidents (2025)")
plt.ylabel("Number of Incidents")
plt.xlabel("Station")
plt.xticks(rotation=45, ha="right")
plt.tight_layout()
plt.savefig("top_stations_by_incidents.png")
plt.show()

# ------------------------------------------------------------
# 3. Question 2: Which stations have the worst TOTAL delay time?
# A station can have few incidents but long delays each time.
# ------------------------------------------------------------
total_delay_by_station = df.groupby("Station")["Min Delay"].sum().sort_values(ascending=False).head(10)
print("\nTop 10 stations by total minutes delayed:")
print(total_delay_by_station)

total_delay_by_station.plot(kind="bar", figsize=(10, 5), color="crimson")
plt.title("Top 10 Stations by Total Delay Minutes (2025)")
plt.ylabel("Total Minutes Delayed")
plt.xlabel("Station")
plt.xticks(rotation=45, ha="right")
plt.tight_layout()
plt.savefig("top_stations_by_total_delay.png")
plt.show()

# ------------------------------------------------------------
# 4. Question 3: Which day of the week has the most delays?
# ------------------------------------------------------------
delays_by_day = df.groupby("Day").size().sort_values(ascending=False)
print("\nDelay incidents by day of week:")
print(delays_by_day)

delays_by_day.plot(kind="bar", figsize=(8, 5), color="steelblue")
plt.title("Delay Incidents by Day of Week (2025)")
plt.ylabel("Number of Incidents")
plt.tight_layout()
plt.savefig("delays_by_day.png")
plt.show()

# ------------------------------------------------------------
# 5. Question 4: Which subway line has the most delays?
# ------------------------------------------------------------
delays_by_line = df.groupby("Line").size().sort_values(ascending=False)
print("\nDelay incidents by line:")
print(delays_by_line)

# ------------------------------------------------------------
# Findings
# ------------------------------------------------------------
# - Bloor Station had the highest number of delay incidents.
# - Eglinton Station had the highest total delay time, despite fewer
#   incidents than Bloor — suggesting longer average delays there.
# - Thursday was the peak day for delays.
# - Line 1 (Yonge-University) had the most delay incidents of any line,
#   consistent with it being TTC's busiest and oldest subway line.
