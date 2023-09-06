#!/usr/bin/env python
# coding: utf-8

import dash
from dash import dcc, html
from dash.dependencies import Input, Output
import pandas as pd
import plotly.express as px

# Load the data using pandas
data = pd.read_csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DV0101EN-SkillsNetwork/Data%20Files/historical_automobile_sales.csv')

# Initialize the Dash app
app = dash.Dash(__name__)

# Set the title of the dashboard
app.title = "Automobile Statistics Dashboard"

# Create the dropdown menu options
dropdown_options = [
    {'label': 'Yearly Statistics', 'value': 'Yearly Statistics'},
    {'label': 'Recession Period Statistics', 'value': 'Recession Period Statistics'}
]

# List of years
year_list = [i for i in range(1980, 2024, 1)]

# Create the layout of the app
app.layout = html.Div([
    # Add title to the dashboard
    html.H1("Automobile Sales Statistics Dashboard", style={'textAlign': 'center', 'color': '#503D36', 'fontSize': 24}),
    html.Div([
        html.Label("Select Statistics:"),
        dcc.Dropdown(
            id='dropdown-statistics',
            options=dropdown_options,
            value='Yearly Statistics',
            placeholder='Select a report type',
            style={'textAlign': 'center', 'width': 80, 'padding': 3, 'fontSize': 20}
        )
    ]),
    html.Div(dcc.Dropdown(
        id='select-year',
        options=[{'label': i, 'value': i} for i in year_list],
        value='Select Year'
    )),
    html.Div([
        html.Div(id='output-container', className='chart-grid', style={'display': 'flex'}),
    ])
])

# Define the callback function to update the input container based on the selected statistics
@app.callback(
    Output(component_id='select-year', component_property='disabled'),
    Input(component_id='dropdown-statistics', component_property='value')
)
def update_input_container(value):
    if value == 'Yearly Statistics':
        return False
    else:
        return True

# Callback for plotting
@app.callback(
    Output(component_id='output-container', component_property='children'),
    [Input(component_id='select-year', component_property='value'), Input(component_id='dropdown-statistics', component_property='value')]
)
def update_output_container(selected_year, selected_statistics):
    if selected_statistics == 'Recession Period Statistics':
        # Filter the data for Recession Statistics
        recession_data = data[data['Recession'] == 1]

        # Create and display graphs for Recession Report Statistics
        # Plot 1: Automobile sales fluctuate over Recession Period (year-wise)
        yearly_rec = recession_data.groupby('Year')['Automobile_Sales'].mean().reset_index()
        R_chart1 = dcc.Graph(
            figure=px.line(
                yearly_rec,
                x='Year',
                y='Automobile Sales',
                title="Average Automobile Sales Fluctuation Over Recession Period",
                labels={'Year': 'Year', 'Automobile Sales': 'Sales'}
            )
        )

        # Plot 2: Calculate the average number of vehicles sold by vehicle type
        average_sales = recession_data.groupby('Vehicle_Type')['Number_Sold'].mean().reset_index()
        R_chart2 = dcc.Graph(
            figure=px.bar(
                average_sales,
                x='Vehicle_Type',
                y='Number_Sold',
                title='Average Vehicles Sold by Vehicle Type During Recession',
                labels={'Vehicle_Type': 'Vehicle Type', 'Number_Sold': 'Average Sales'}
            )
        )

        # Plot 3: Pie chart for total expenditure share by vehicle type during recessions
        exp_rec = recession_data.groupby('Vehicle_Type')['Advertisement_Expenditure'].sum().reset_index()
        R_chart3 = dcc.Graph(
            figure=px.pie(
                exp_rec,
                names='Vehicle_Type',
                values='Advertisement_Expenditure',
                title='Total Expenditure Share by Vehicle Type During Recession'
            )
        )

        # Plot 4: Bar chart for the effect of unemployment rate on vehicle type and sales
        unemployment_data = recession_data.groupby('Vehicle_Type')['Unemployment_Rate'].mean().reset_index()
        R_chart4 = dcc.Graph(
            figure=px.bar(
                unemployment_data,
                x='Vehicle_Type',
                y='Unemployment_Rate',
                title='Effect of Unemployment Rate on Vehicle Type and Sales During Recession',
                labels={'Vehicle_Type': 'Vehicle Type', 'Unemployment_Rate': 'Average Unemployment Rate'}
            )
        )

        return [
            html.Div(className='chart-item', children=[html.Div(children=R_chart1), html.Div(children=R_chart2)]),
            html.Div(className='chart-item', children=[html.Div(children=R_chart3), html.Div(children=R_chart4)])
        ]

    elif selected_statistics == 'Yearly Statistics' and selected_year != 'Select Year':
        yearly_data = data[data['Year'] == int(selected_year)]

        # Create and display graphs for Yearly Report Statistics
        # Plot 1: Yearly Automobile sales using a line chart for the whole period
        y_as = yearly_data.groupby('Year')['Automobile_Sales'].mean().reset_index()
        Y_chart1 = dcc.Graph(
            figure=px.line(
                y_as,
                x='Year',
                y='Automobile Sales',
                title=f"Yearly Automobile Sales for the Whole Period ({selected_year})",
                labels={'Year': 'Year', 'Automobile Sales': 'Sales'}
            )
        )

        # Plot 2: Total Monthly Automobile sales using a line chart
        m_as = yearly_data.groupby('Month')['Automobile_Sales'].sum().reset_index()
        Y_chart2 = dcc.Graph(
            figure=px.line(
                m_as,
                x='Month',
                y='Automobile Sales',
                title=f"Total Monthly Automobile Sales ({selected_year})",
                labels={'Month': 'Month', 'Automobile Sales': 'Sales'}
            )
        )

        # Plot 3: Bar chart for the average number of vehicles sold during the given year
        avg_vdata = yearly_data.groupby('Vehicle_Type')['Automobile_Sales'].mean().reset_index()
        Y_chart3 = dcc.Graph(
            figure=px.bar(
                avg_vdata,
                x='Vehicle_Type',
                y='Automobile_Sales',
                title=f'Average Vehicles Sold by Vehicle Type in {selected_year}',
                labels={'Vehicle_Type': 'Vehicle Type', 'Automobile_Sales': 'Average Sales'}
            )
        )

        # Plot 4: Pie chart for total Advertisement Expenditure for each vehicle
        exp_data = yearly_data.groupby('Vehicle_Type')['Advertisement_Expenditure'].sum().reset_index()
        Y_chart4 = dcc.Graph(
            figure=px.pie(
                exp_data,
                names='Vehicle_Type',
                values='Advertisement_Expenditure',
                title=f'Total Advertisement Expenditure Share by Vehicle ({selected_year})'
            )
        )

        return [
            html.Div(className='chart-item', children=[html.Div(children=Y_chart1), html.Div(children=Y_chart2)]),
            html.Div(className='chart-item', children=[html.Div(children=Y_chart3), html.Div(children=Y_chart4)])
        ]

    else:
        return None

# Run the Dash app
if __name__ == '__main__':
    app.run_server(debug=True)