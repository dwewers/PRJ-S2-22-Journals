## Journal 4 - 24/08/2022

### React component

During the learning of the Power Apps component, the main way that most tutorials or documentation showed was the use of TypeScript with creating each element and defining its classes and inner html manually, or returning a full html script as a string with the parameters passed in where needed. An example of this is as shown below:

Option A:

```typescript
// Required private parameters of the control
private _context: ComponentFramework.Context<IInputs>;
private _container: HTMLDivElement;
private _main: HTMLDivElement;

public init(context: ComponentFramework.Context<IInputs>, notifyOutputChanged: () => void, state: ComponentFramework.Dictionary, container:HTMLDivElement)
{
    // assigning environment variables.
    this._context = context;
    this._container = container;
    this._main = document.createElement("div");
    this._creditCardErrorElement.setAttribute("class", "main-div");
    //And any other details
    
    this._container.appendChild(this._main)
}
```

Option B:

```typescript
// Required private parameters of the control
private _context: ComponentFramework.Context<IInputs>;
private _container: HTMLDivElement;
private _main: HTMLDivElement;

public init(context: ComponentFramework.Context<IInputs>, notifyOutputChanged: () => void, state: ComponentFramework.Dictionary, container:HTMLDivElement)
{
    // assigning environment variables.
    this._context = context;
    this._container = container;
    this._main = document.createElement("div");
    //And any other details
    var html =
       `<div class="main-div">
            <div class="full-name">
                <h1>${firstName} ${lastName}</h1>
            </div>
        </div>`
    this._main.innerHTML = html;
    
    this._container.appendChild(this._main)
}
```

This would not be an issue for a small component, but for a component as complex as the one for this project, the code becomes overly complex, or a long string of HTML which is very messy. Microsoft recently added framework compatibility for the Power App Component Framework, including React. This was a significant discovery during the project's development because it allows for better separation of concerns as well as cleaner and more reusable code. This means that we can now use the latest Microsoft Power Platform CLI for model-driven apps, canvas apps, and portals to create code components directly with the React JS and Fluent UI libraries without any custom project setup. 

Similar to the pac CLI command used for a default component, we type out the same details (name, namespace, and template), however with the addition of a framework parameter. The following is an example of this:

```powershell
pac pcf init --name SummaryComponent --namespace Fuseit.S4D.Dynamics --template field --framework React
```

In the `index.ts` file, we have `updateView()` method where we are initializing the React component where you can pass your own component as an entry point for the other React Component. This is shown below:

```typescript
//index.ts
import {LeadSummaryComponent, IStats} from "./LeadSummaryComponent"

public updateView(context: ComponentFramework.Context<IInputs>): React.ReactElement{
	const props: IStats = {
        name:"Daniel",
        age: 21
    }
    return React.createElement(
		LeadSummaryComponent, props
	);
}

//LeadSummaryComponent.tsx
export interface IStats{
    name: string,
    age: number
}

export class LeadSummaryComponent extends React.Component<IStats> {
    public render(): React.ReactNode {
        return (
        	<div class="main-div">
            	<h1>{this.props.name}</h1>
            </div>
        )
    }
}
```

Below is a comparison between the project component using the old method versus using the React framework:

Old Method:



React Framework:

```react
import * as React from 'react';
import { TextField } from '@fluentui/react/lib/TextField';
import { DefaultButton } from '@fluentui/react/lib/Button';
import { IInputs } from "./generated/ManifestTypes";

import GetData from './api/data';

export interface ISummaryStats {
  goalValue: number,
  visitCount: number,
  averageVisitTime: number,
  pageViews: number,
  firstVisit: string,
  latestVisit: string,
  leadid: string,
  email: string,
  firstName: string,
  lastName: string,
  gender: string,
  language: string,
  visits: Array<any>,
  context: ComponentFramework.Context<IInputs>,
}

export class SummaryComponent extends React.Component<ISummaryStats, ISummaryStats> {

  constructor(props: ISummaryStats) {
    super(props);
    this.state = { ...props };
  }

  refreshData = async () => {
    let data = await GetData(this.state.context).then((res:any) => {
      return res;
    });
    this.setState({ ...data });
  }

  public render(): React.ReactNode {
    return (
      <div className="summary-container">
        <div className="row">
          <br />
        </div>
        <div className="row">
          <div className="col-md-12">
            <div className="card">
              {/* <!-- Main Card --> */}
              <div className="card-body">
                <h3 className="card-title">Sitecore CDP Guest</h3>
                {/* <!-- Overview --> */}
                <div className="card">
                  <div className="card-body">
                    <div className="row">
                      {/* <!-- Goal Value --> */}
                      <div className="col-md-3 text-center">
                        <div className="h1">
                          {this.state.goalValue}
                        </div>
                        <div className="h6">Goal Value</div>
                      </div>
                      {/* <!-- Visit Count --> */}
                      <div className="col-md-3 text-center">
                        <div className="h1">
                          {this.state.visitCount}
                        </div>
                        <div className="h6">Visit Count</div>
                      </div>
                      {/* <!-- Avg Visit Time --> */}
                      <div className="col-md-3 text-center">
                        <div className="h1">
                          {this.state.averageVisitTime}
                        </div>
                        <div className="h6">Avg Visit Time</div>
                      </div>
                      {/* <!-- Views --> */}
                      <div className="col-md-3 text-center">
                        <div className="h1">
                          {this.state.pageViews}
                        </div>
                        <div className="h6">Views</div>
                      </div>
                    </div>
                    <br />
                    {/* <!-- Refresh Analytics --> */}
                    <div className="row">
                      <div className="col-md-12 text-center">
                        <DefaultButton text='Refresh Analytics' allowDisabledFocus onClick={this.refreshData}/>
                      </div>
                    </div>
                  </div>
                </div>
                <hr />
                {/* <!-- Details --> */}
                <div className="card">
                  <div className="card-body">
                    {/* <!-- First and last visit --> */}
                    <div className="row">
                      <div className="col-md-6">
                        <TextField label="First Visit" id="firstVisit" value={this.state.firstVisit} readOnly />
                      </div>
                      <div className="col-md-6">
                        <TextField label="Latest Visit" id="lastVisit" value={this.state.latestVisit} readOnly />
                      </div>
                    </div>
                    {/* <!-- Reference and Email --> */}
                    <div className="row">
                      <div className="form-group col-md-6">
                        <TextField label="Reference" id="reference" value={this.state.leadid} readOnly />
                      </div>
                      <div className="form-group col-md-6">
                        <TextField label="Email" id="email" value={this.state.email} readOnly />
                      </div>
                    </div>
                    {/* <!-- First and last name --> */}
                    <div className="row">
                      <div className="form-group col-md-6">
                        <TextField label="First Name" id="firstName" value={this.state.firstName} readOnly />
                      </div>
                      <div className="form-group col-md-6">
                        <TextField label="Last Name" id="lastName" value={this.state.lastName} readOnly />
                      </div>
                    </div>
                    {/* <!-- Gender and language --> */}
                    <div className="row">
                      <div className="form-group col-md-6">
                        <TextField label="Gender" id="gender" value={this.state.gender} readOnly />
                      </div>
                      <div className="form-group col-md-6">
                        <TextField label="Language" value={this.state.language} readOnly />
                      </div>
                    </div>
                  </div>
                </div>
                <br />
                <div className="h5">Sitecore CDP Session</div>
                {/* <!-- Visits Table --> */}
                <div className="card">
                  <div className="card-body">
                    <div className="table-responsive">
                      <table className="table table-striped">
                        <thead>
                          <tr>
                            <th scope="col">Duration</th>
                            <th scope="col">Date</th>
                            <th scope="col">Value</th>
                            <th scope="col">Page Views</th>
                            <th scope="col">Traffic Channel</th>
                            <th scope="col">Browser Name</th>
                          </tr>
                        </thead>
                        <tbody id="table-contents">
                          {this.state.visits.map((visit, index) => {
                            return (
                              <tr key={index}>
                                <td>{visit.duration}</td>
                                <td>{visit.date}</td>
                                <td>{visit.value}</td>
                                <td>{visit.pageViews}</td>
                                <td>{visit.trafficChannel}</td>
                                <td>{visit.browserName}</td>
                              </tr>
                            )
                          })}
                        </tbody>
                      </table>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    )
  }
}

```

Overall, this was a big week in the development of the project.
