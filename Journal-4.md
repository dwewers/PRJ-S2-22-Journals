## Journal 4 - 17/08/2022

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

Below is a comparison between the a component using the with and without using the React framework:

Without React Framework:

```typescript
public init(context: ComponentFramework.Context<IInputs>, notifyOutputChanged: () => void, state: ComponentFramework.Dictionary, container:HTMLDivElement)
{
    this._container = container;
    this._main = document.createElement("div");
    
    var html =
           `
              <div className="container">
                <div className="row">
                  <div className="col-md-12">
                     ${ this.props.name }
                  </div>
                </div>
              </div>
            `
    this._main.innerHTML = html;
    
    this._container.appendChild(this._main)
}
```

With React Framework:

```React
export class SummaryComponent extends React.Component<ISummaryStats, ISummaryStats> {
  	constructor(props){
        super(props);
    }
    
    public render(): React.ReactNode {
    return (
      <div className="container">
        <div className="row">
          <div className="col-md-12">
             { this.props.name }
          </div>
        </div>
      </div>
    )
  }
}
```

Overall, this was a big week in the development of the project.
