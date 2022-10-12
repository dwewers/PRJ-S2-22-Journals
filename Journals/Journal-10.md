# Journal 10 - 28/09/2022

This week some further work was done on standardisation. Last week the table was changed from a normal HTML/TSX table into a FluentUI table. However, the rest of the layout was still using normal `div's` to created the components layout. After doing some research, it was discovered that FluentUI has a component called a `Stack` (parent element) that has sub components called `Stack.Item` (child element). These work in a similar way to that of grid CSS.

You simply create a `Stack` like you would a `div` and add `Stack.Item`'s to them':

```typescript
<Stack>
	<Stack.Item></Stack.Item>
</Stack>
```

These are useful as you can define whether the stack goes horizontally or vertically. After a bit of time, the following was created:

```typescript
<Stack>
    {/* Main title */}
    <StackItem style={styles.title}>
        Sitecore Guest
    </StackItem>
    {/* Sitecore Summary Sectionn */}
    <StackItem style={styles.section}>
        <Stack horizontal horizontalAlign='center'>
            <StackItem>
                <Stack style={styles.secondary}>
                    <StackItem style={styles.header}>
                        {this.props.Statistics.value}
                    </StackItem>
                    <StackItem style={styles.subHeader}>
                        Goal Value
                    </StackItem>
                </Stack>
            </StackItem>
            <StackItem>
                <Stack style={styles.secondary}>
                    <StackItem style={styles.header}>
                        {this.props.Statistics.count}
                    </StackItem>
                    <StackItem style={styles.subHeader}>
                        Visit Count
                    </StackItem>
                </Stack>
            </StackItem>
            <StackItem>
                <Stack style={styles.secondary}>
                    <StackItem style={styles.header}>
                        {this.props.Statistics.duration}
                    </StackItem>
                    <StackItem style={styles.subHeader}>
                        Avg Visit Time
                    </StackItem>
                </Stack>
            </StackItem>
            <StackItem>
                <Stack style={styles.secondary}>
                    <StackItem style={styles.header}>
                        {this.props.Statistics.views}
                    </StackItem>
                    <StackItem style={styles.subHeader}>
                        Views
                    </StackItem>
                </Stack>
            </StackItem>
        </Stack>
        <Stack horizontal horizontalAlign='center'>
            {/* Refresh Analytics Button */}
            <StackItem style={styles.button}>
                <PrimaryButton text='Refresh Analytics' onClick={this.refresh} />
            </StackItem>
        </Stack>
        <StackItem align="auto">
            <Spinner size={SpinnerSize.large} label="Refreshing Analytics..." style={{display: display}} />
        </StackItem>
    </StackItem>

    {/* Secondary Title */}
    <StackItem style={styles.subTitle}>
        Sitecore Sessions
    </StackItem>
    {/* List of visits */}
    <StackItem style={styles.section}>
        <VisitsDetailsList data={this.props.Statistics.visits} />
    </StackItem>
</Stack>
```

As you can see, each of the components have their own style relative to what they are. Rather than doing inline styling, we can create style objects that have a set of defined styles. For example:

```typescript
const styles = {
    title: {
        "color": palette.midBlue,
        "fontSize": "2.25rem",
        "fontWeight": "bold",
        "margin": "0.5rem",
        "padding": "0.5rem",
    },
    subTitle: {
        "color": palette.midBlue,
        "fontSize": "1.5rem",
        "fontWeight": "bold",
        "margin": "0.5rem",
        "padding": "0.5rem",
    },
    header: {
        "color": palette.darkBlue,
        "fontSize": "1.75rem",
        "fontWeight": "bold"
    },
    subHeader: {
        "color": palette.darkBlue,
        "fontSize": "1.5rem",
        "fontWeight": "lighter"
    },
    section: {
        "border": "3px solid" + palette.lightBlue,
        "padding": "10px",
        "margin": "10px",
    },
    secondary: {
        "border": "3px solid" + palette.lightGrey,
        "padding": "10px",
        "margin": "10px",
    },
    button: {
        "margin": "1rem",
    }
}
```

Additionally, colour palettes can be created in the same way  and used throughout the style objects. The colour palette shown below is the same that is used in Microsoft Dynamics:

```typescript
const palette = {
    darkBlue: "#002050",
    midBlue: "#1160b7",
    lightBlue: "#b1d6f0",
    lightGrey: "#dfe2e8",
    red: "#d24726"
}
```

As this weeks focus was more on the standardising of the components, not a lot was done in other areas. This was mainly due to working from home and needing to spend some time tidying up the component before further work was done on the plugins. However, with what was achieved this week, there isn't much left to do component side, just plugin side.
