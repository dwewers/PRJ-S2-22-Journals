Standardised
import * as React from 'react';
import { IInputs, IOutputs } from "../generated/ManifestTypes";
import { initializeIcons, Stack, StackItem, PrimaryButton, IStackTokens } from '@fluentui/react';
import { VisitsDetailsList, IVisitsProps } from "./DetailsList";

initializeIcons();

export interface ISitecoreStatistics {
    id: string,
    status: string,
    contactAliasId: string,
    count: number,
    duration: number,
    views: number,
    value: number,
    visits: Array<ISitecoreVisit>
}

export interface ISitecoreVisit {
    id: string,
    interactionId: string,
    views: number,
    browser: string,
    value: number,
    channel: string
}

export interface IProps {
    Statistics: ISitecoreStatistics,
    Config: IConfig,
    Context: ComponentFramework.Context<IInputs>
}

interface IConfig  {
    SitecoreApiUrl: string,
    SitecoreApiKey: string,
    maxVisits: number
}

// interface IState {
//     Count: number,
//     Views: number,
//     Value: number,
//     Duration: number,
//     Visits: Array<ISitecoreVisit>
// }



export const ComponentBase = ({ Statistics, Config, Context }: IProps): JSX.Element => {
    const refresh = () => {
        let request = {
            // Input parameters.
            leadid: Statistics.id,
            // Metadata for calling custom unbound action.
            getMetadata: function () {
                return {
                    boundParameter: null,
                    operationType: 0,
                    operationName: 'dev_RefreshAnalytics',
                    parameterTypes: {
                        'leadid': {
                            typeName: 'Edm.String',
                            structuralProperty: 1,
                        },
                    },
                };
            },
        };

        Xrm.WebApi.online.execute(request).then(
            (result: any) => {
                // On Success
                alert("Action Executed");
                // Refresh the component
                Context.factory.requestRender();
            },
            (error: any) => {
                alert(error.message);
            }
        );
    };

    return (
        <Stack>
            {/* Main title */}
            <StackItem style={styles.title}>
                Sitecore CDP Guest
            </StackItem>
            {/* Sitecore Summary Sectionn */}
            <StackItem style={styles.section}>
                <Stack horizontal horizontalAlign='center'>
                    <StackItem>
                        <Stack style={styles.secondary}>
                            <StackItem style={styles.header}>
                                {Statistics.value}
                            </StackItem>
                            <StackItem style={styles.subHeader}>
                                Goal Value
                            </StackItem>
                        </Stack>
                    </StackItem>
                    <StackItem>
                        <Stack style={styles.secondary}>
                            <StackItem style={styles.header}>
                                {Statistics.count}
                            </StackItem>
                            <StackItem style={styles.subHeader}>
                                Visit Count
                            </StackItem>
                        </Stack>
                    </StackItem>
                    <StackItem>
                        <Stack style={styles.secondary}>
                            <StackItem style={styles.header}>
                                {Statistics.duration}
                            </StackItem>
                            <StackItem style={styles.subHeader}>
                                Avg Visit Time
                            </StackItem>
                        </Stack>
                    </StackItem>
                    <StackItem>
                        <Stack style={styles.secondary}>
                            <StackItem style={styles.header}>
                                {Statistics.views}
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
                        <PrimaryButton text='Refresh Analytics' onClick={refresh} />
                    </StackItem>
                </Stack>
            </StackItem>

            {/* Secondary Title */}
            <StackItem style={styles.subTitle}>
                Sitecore CDP Sessions
            </StackItem>
            {/* List of visits */}
            <StackItem style={styles.section}>
                <VisitsDetailsList data={Statistics.visits} />
            </StackItem>
        </Stack>
    );
};

const palette = {
    darkBlue: "#002050",
    midBlue: "#1160b7",
    lightBlue: "#b1d6f0",
    lightGrey: "#dfe2e8",
    red: "#d24726"
}


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
        "border":"3px solid" + palette.lightBlue,
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



// TODO:
// 1. Add a error handling for the fetch call
// 2. Add a loading indicator
// 3. Add a error message if the fetch call fails
// 4. Add a error message if the fetch call returns no data
// 5. Add a duration for how long the message should be displayed (5 seconds)
// 6. Add a refresh button to contact the Sitecore
// 7. Maybe success? :)

//! Error Example:
/*
    const ErrorExample = (p: IExampleProps) => (
    <MessageBar
        messageBarType={MessageBarType.error}
        isMultiline={false}
        onDismiss={p.resetChoice}
        dismissButtonAriaLabel="Close"
    >
        Error MessageBar with single line, with dismiss button.
        <Link href="www.bing.com" target="_blank" underline>
        Visit our website.
        </Link>
    </MessageBar>
    );
*/

//* Success Example:
/*
    const SuccessExample = () => (
    <MessageBar
        actions={
        <div>
            <MessageBarButton>Yes</MessageBarButton>
            <MessageBarButton>No</MessageBarButton>
        </div>
        }
        messageBarType={MessageBarType.success}
        isMultiline={false}
    >
        Success MessageBar with single line and action buttons.
        <Link href="www.bing.com" target="_blank" underline>
        Visit our website.
        </Link>
    </MessageBar>
    );
*/

//! ANCHOR: ERROR CASES:
//! 1. Fetch call fails
//! 2. Fetch call returns no data
//! 3. Fetch call returns data but the data is not in the correct format
//! 4. Guest does not exist
//! 5. Guest exists but has no data
//! 6. Id is invalid syntax (########-####-####-####-############) EX. (7ba18ae0-4d0e-ea11-a813-000d3a1bbd52)
//! 7. Id is valid syntax but does not exist in Sitecore
