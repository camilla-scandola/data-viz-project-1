import pandas as pd
import plotly.graph_objects as go
import plotly.express as px
from plotly.subplots import make_subplots

# Load data
data = pd.read_csv("KW Cat with location - All Data.csv")

# Check for Location in dataset
if "Location" in data.columns:
    location_counts = data["Location"].value_counts()

# Consider only the top locations (Canada + main cities + "Others")
top_n = 10
top_locations = location_counts.nlargest(top_n)
others_sum = location_counts.iloc[top_n:].sum()
others = pd.Series({"Others": others_sum})
top_locations = pd.concat([top_locations, others])

# Colors
custom_colors = [
        "#FF92B2",  # canada
        "#EECFFC",  # toronto
        "#F0E9F7",  # montreal
        "#F7D5C2",  # online
        "#FAD1D1",  # vancouver
        "#F8FFBF",  # calgary
        "#F8B3D0",  # ottawa
        "#FDF5F5",  # edmonton
        "#FAD1D1",  # brampton
        "#FFCCBC",  # others
        "#FFCCBC",  # mississauga
        "#FAD1D1",  # winnipeg
    ]

custom_colors_shifts = ["#FF92B2", #increased
                        "#EECFFC", #stable
                        "#F7D5C2" #decreased
                        ]

# Create subplots for each graph
fig = make_subplots(
    rows=2, cols=2, 
    subplot_titles=(
        "Keyword share across Canada", 
        "SERP performance breakdown in Canada and by city",
        "KW position changes by ranking group (Top 3, Top 10, etc.)", 
        "Record Count by Location and position shifts"
    ),
    specs=[
        [{'type': 'domain'}, {'type': 'xy'}],  # Pie chart in (1,1), Bar chart in (1,2)
        [{'type': 'xy'}, {'type': 'xy'}]       # Stacked Bar in (2,1), Line chart in (2,2)
    ],
    horizontal_spacing=0.15, vertical_spacing=0.2
)

# Pie Chart - Keyword share across Canada
fig.add_trace(
    go.Pie(labels=top_locations.index, values=top_locations.values, hole=0.6, marker=dict(colors=custom_colors)),
    row=1, col=1
)

# Bar Chart - SERP performance breakdown in Canada and by city
serp_position_color_map = {
    "Top 3": "#FF92B2", # Color Map for SERP positions
    "Top 10": "#EECFFC",
    "Everything Else": "#FFCCBC",
}
if "serp_position" in data.columns:
    filter_data = data[data["Location"].isin(top_locations.index)]
    serp_ranking = filter_data.groupby(["Location", "serp_position"], as_index=False).size()
    serp_ranking.rename(columns={"size": "count"}, inplace=True)

    category_order = ["canada", "toronto", "montreal", "vancouver", "online", "ottawa", "calgary", "mississauga",
                      "brampton", "winnipeg"]
    serp_ranking["Location"] = pd.Categorical(serp_ranking["Location"], categories=category_order, ordered=True)
    serp_shift = serp_ranking.sort_values("Location")

    for pos in serp_ranking["serp_position"].unique():
        color = serp_position_color_map.get(pos, "#FFCCBC")  # Default to a fallback color
        subset = serp_ranking[serp_ranking["serp_position"] == pos]
        fig.add_trace(go.Bar(x=subset["Location"],
                             y=subset["count"],
                             name=str(pos),
                             marker=dict(color=color)),
                             row=1, col=2)


# Stacked Bar Chart
if "position_shifts" in data.columns:
    filter_data = data[data["position_shifts"].notna()]
    serp_shift = filter_data.groupby(["position_shifts", "serp_position"], as_index=False).size()
    serp_shift.rename(columns={"size": "count"}, inplace=True)

    for shift, color in zip(serp_shift["position_shifts"].unique(), custom_colors_shifts):
        subset = serp_shift[serp_shift["position_shifts"] == shift]
        fig.add_trace(go.Bar(x=subset["count"],
                             y=subset["serp_position"],
                             name=str(shift),
                             orientation='h',
                             marker=dict(color=color)),
                             row=2, col=1)

# Line Chart
if "position_shifts" in data.columns:
    filter_data = data[data["Location"].isin(top_locations.index)]
    serp_shift = filter_data.groupby(["position_shifts", "Location"], as_index=False).size()
    serp_shift.rename(columns={"size": "count"}, inplace=True)

    category_order = ["canada", "toronto", "montreal", "vancouver", "online", "ottawa", "calgary", "mississauga", "brampton", "winnipeg"]
    serp_shift["Location"] = pd.Categorical(serp_shift["Location"], categories=category_order, ordered=True)
    serp_shift = serp_shift.sort_values("Location")

    for shift, color in zip(serp_shift["position_shifts"].unique(), custom_colors_shifts):
        subset = serp_shift[serp_shift["position_shifts"] == shift]
        fig.add_trace(go.Scatter(x=subset["Location"],
                                 y=subset["count"],
                                 mode='lines+markers',
                                 name=str(shift),
                                 line=dict(color=color)),
                                 row=2, col=2)

# Update layout
fig.update_layout(height=900, width=1200, showlegend=True, title_text="Data Viz Project - Keyword Performance")
fig.show()
