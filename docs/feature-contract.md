# Feature contract

All named columns are required in CSVs. A blank value means missing; extra columns are ignored. Defaults are synthetic training medians/modes. Numeric bounds constrain serving and mark unsupported source values missing during training. They do not define a population validity guarantee.

| Source field | Meaning / units | Accepted values |
|---|---|---|
| `Delivery_person_Age` | Delivery person Age | 18 to 80 |
| `Delivery_person_Ratings` | Delivery person Ratings | 1 to 5 |
| `distance_km` | Great-circle distance (km) | 0 to 300 |
| `Weather_conditions` | Weather conditions | Cloudy, Fog, Sandstorms, Stormy, Sunny, Windy |
| `Road_traffic_density` | Road traffic density | High, Jam, Low, Medium |
| `Vehicle_condition` | Vehicle condition | 0 to 3 (whole numbers) |
| `Type_of_order` | Type of order | Buffet, Drinks, Meal, Snack |
| `Type_of_vehicle` | Type of vehicle | bicycle, electric_scooter, motorcycle, scooter |
| `multiple_deliveries` | multiple deliveries | 0 to 5 (whole numbers) |
| `Festival` | Festival | No, Yes |
| `City` | City | Metropolitian, Semi-Urban, Urban |
