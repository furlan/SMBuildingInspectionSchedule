# Santa Monica City - Building Inspection Schedule Visualizer

This documentation is about the visualization app for Santa Monica Building Inspection Schedule.

## Business Case

The team of inspectors from Santa Monica City is struggling to manage the increasing number of building inspections required by the public. 

Helping frontline workers complete their day-to-day tasks is a common business scenario. The City of Santa Monica publishes their [Building Permit Inspection Schedule as open data](https://data.smgov.net/Permits-Licenses/Permit-Inspections-Schedule/xird-2kxi). Building inspectors need to travel to various locations throughout the day to inspect a building under construction. Building inspector supervisors need to ensure timely delivery of services and analyze overall trends. Please build something that helps the Building and Safety Division manage their requests for building permit inspections.

This application provides a way for the inspectors to visualize the coming inspections and check inspection details and locations on the road.

## User Guide

The application is composed of three screens. The first screen presented to the user is the Inspections by Inspectors.

You can navigate back to the previous page by tapping on the back arrow on the left top of the page. 

### Inspections by Inspectors

This screen comprises a list of all inspectors scheduled from the date selected from the top of the page. 

As soon you pick one inspector, the list of all inspections is loaded on the right side.

![](/pics/scr1_overview.png)

You can also filter by inspection type using the filter at the top of the page.

![](/pics/scr1_inspection_type_filter.png)

When you pick one inspection, the Inspection Detail page is opened.

### Inspection Detail

This screen is to visualize the details of a selected inspection. 

![](/pics/scr2_overview.png)

You can scroll up and down if the description is too big for the reserved space.

You can zoom in/out of the map to visualize the inspection site better.

To list all the inspections for the same Permit Number, you can tap on the button "Related Inspections" to open the screen Related Inspections.

### Related Inspections

This screen is to visualize all inspections for a selected Permit Number. 

![](/pics/scr3_overview.png)

When you pick one inspection, the Inspection Detail page is opened.

## Technical Solution

### Overview

This app was built using the PowerApps Platform. As I'm using the public API, it's only for data visualization. The app does not make any change in the data.

### Public API Usage

The public API is very flexible, so I created a Generic Custom Connector. The Inspections Generic Query connector is responsible for making calls to the API using only one parameter: ```$query```. It gave all the flexibility to retrieve the data necessary for the application.

![](/pics/gen_conn_request.png)

As per [API documentation](https://dev.socrata.com/docs/app-tokens.html), I'm using an App Token to avoid any throttling limits. The key is passed in the header of the call with the parameter ```X-App-Token```. 

To be transparent to the user, I marked this parameter as internal, so I don't need to include the token in every connector call. Also, it's an only point to change in case it's necessary to change the key.

![](/pics/gen_conn_x-app-token.png)

### Loading Data

Here is the list of all data retrieving and the respective query string.

#### Inspectors List

- Triggered when the app is oppened.
- List all inspectors for the date from field.

```C#
InspectionsGenericQuery.GetInspectionsByQuery(
    Concatenate(
        "select distinct inspector_assigned WHERE inspection_scheduled_date > '",
        Text(
            FromDate.SelectedDate,
            "yyyy-mm-ddThh:mm:ss"
        ),
        "' AND inspector_assigned <> ''"
    )
)
```

#### Inspections by Inspectors

- Triggered when one inspector is selected.
- List all inspections for the inspector selected and for the date from field.
- List all inspections type for the inspector selected and for the date from field.
    - to be used on the dropdown list on the top of the screen Inspections by Inspector.

```C#
Set(
    SelectedInspector,
    Gallery1.Selected.inspector_assigned
);
With(
    {
        DateFilter: Text(
            Now() - 7,
            "yyyy-mm-ddThh:mm:ss"
        )
    },
    Set(
        Inspections2,
        InspectionsGenericQuery.GetInspectionsByQuery(
            Concatenate(
                "select * where inspection_scheduled_date>'",
                DateFilter,
                "' and inspector_assigned='",
                SelectedInspector,
                "'"
            )
        )
    );
    Set(
        InspectionsTypes,
        InspectionsGenericQuery.GetInspectionsByQuery(
            Concatenate(
                "select distinct inspection_type where inspection_scheduled_date>'",
                DateFilter,
                "' and inspector_assigned='",
                SelectedInspector,
                "'"
            )
        )
    );
    
)
```

#### Related Inspections

- Triggered when one inspection is selected.
- List all inspections for the same ```permit_number``.
    - to be used in the list of the screen Related Inspections.

```C#
Set(
    Inspection,
    ThisItem
);
Set(
    RelatedInspections,
    InspectionsGenericQuery.GetInspectionsByQuery(
        Concatenate(
            "SELECT * WHERE permit_number = '",
            Inspection.permit_number,
            "'"
        )
    )
);
```

### Road Map

- Map with all the inspections sites for the selected inspector.
- Calendar showing the distribution of the inspections across the days.
- Dashboard showing the distribution of inspections among the inspectors.