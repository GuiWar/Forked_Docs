---
title: 'Go to the next level - Create a form to input data'
sidebar_label: 'Create a form to input data'
id: forms
keywords: [getting started, quick start, next level, forms]
tags:
    - getting started
    - quick start
    - next level
    - forms
---

## Section objectives
The goal of this section is to add a form that enables us to insert trades into our database.

## Interacting with the Event Handler
To interact with the Event Handler that you created [previously](../../../getting-started/go-to-the-next-level/events#event-handler), you will now create a form that will collect the data from the user.

Start with the Form component, which will generate all the inputs based on the API:

```html title='home.template.ts'
<foundation-form
  resourceName="EVENT_TRADE_INSERT"
></foundation-form>
```

To respond to the user clicking on the **Submit** button, you need to call the `EVENT_TRADE_INSERT` event:
```html {3} title='home.template.ts'
<foundation-form
  resourceName="EVENT_TRADE_INSERT"
  @submit=${(x, c) => x.insertTrade(c.event as CustomEvent)}
></foundation-form>
```

Define the `insertTrade` function in the file **home.ts**:

```typescript title='home.ts'
  import {Connect} from '@genesislcap/foundation-comms';
```

```typescript title='home.ts'
  @Connect connect: Connect;

  public async insertTrade(event) {
    const formData = event.detail.payload;
    const insertTradeEvent = await this.connect.commitEvent('EVENT_TRADE_INSERT', {
      DETAILS: {
        COUNTERPARTY_ID: 'GENESIS',
        INSTRUMENT_ID: formData.INSTRUMENT_ID,
        QUANTITY: formData.QUANTITY,
        PRICE: formData.PRICE,
        SIDE: formData.SIDE,
        TRADE_DATETIME: Date.now(),
      },
    });
  }
```

The form is configured as required, but if you look at the page currently you will not see it. Next we need to add some sections and styling to the page. Ensure the template in **home.template.ts** looks like the following.

```html {1,15-19} title='home.template.ts'
<div class="column-split-layout">
	<zero-grid-pro persist-column-state-key="position-grid-settings">
		<grid-pro-genesis-datasource
			resource-name="ALL_POSITIONS"
			order-by="INSTRUMENT_ID"
		></grid-pro-genesis-datasource>
		${repeat(
			() => positionColumnDefs,
			html`
				<grid-pro-column :definition="${(x) => x}"></grid-pro-column>
			`
		)}
		<grid-pro-column :definition="${(x) => x.singlePositionActionColDef}"></grid-pro-column>
	</zero-grid-pro>
	<foundation-form
		resourceName="EVENT_TRADE_INSERT"
		@submit=${(x, c) => x.insertTrade(c.event as CustomEvent)}
	></foundation-form>
</div>
```

And then set the styling required on the `host` element, and the `column-split-layout` div in **home.styles.ts**.
```typescript title='home.styles.ts'
import { css } from '@microsoft/fast-element';
import { mixinScreen } from '../../styles';

export const HomeStyles = css`
  :host {
    ${mixinScreen('flex')}
    align-items: center;
    justify-content: center;
    flex-direction: column;
  }

  .column-split-layout {
    display: flex;
    flex-direction: column;
    flex: 1;
    width: 100%;
  }

  .row-split-layout {
    display: flex;
    flex-direction: row;
    flex: 1;
    width: 100%;
    height: 50%;
  }
`;
```

After refreshing your application, a form should be displayed. The form might sit on top of the grid or by itself, depending on whether you appended to or replaced the already existing markup in **home.template.ts**.

![](/img/trade-insert-form-2023_1.png)

## Adding customisation
What we have done so far is good for simple forms or prototyping, but what if we need much more customisation?
Let's replace the form and code elements above with a more configurable solution.

To do this, you must create each form element manually and take care of storing the data input by the user.

Start by adding the elements to the template. Instead of the `<foundation-form>` above, replace it with the following:

```html title='home.template.ts'
<zero-text-field>Quantity</zero-text-field>
<zero-text-field type="number">Price</zero-text-field>
<span>Instrument</span>
<zero-select></zero-select>
<span>Side</span>
<zero-select></zero-select>
```

Then, define the variables that will hold the values that the user enters:

In the file **home.ts**, add the following properties to the class: `Home`:

```ts {1,4-7} title='home.ts'
import { customElement, FASTElement, observable } from '@microsoft/fast-element';

export class Home extends FASTElement {
  @observable public quantity: string;
  @observable public price: string;
  @observable public instrument: string;
  @observable public side: string = 'BUY';

	...
}
```

Now we need to interact with the Event Handlers that respond to user changes and also store the inputted data:

We can do it in the traditional way by adding `@change` [Event Handler](https://www.fast.design/docs/fast-element/declaring-templates#events) - but we can also use the `sync` directive, which does that for us automatically.

Let's add it to each form element:

```ts title='home.template.ts'
import {sync} from '@genesislcap/foundation-utils';
```

```html {2,7,13,18} title='home.template.ts'
<zero-text-field
  :value=${sync(x=> x.quantity)}
>
  Quantity
</zero-text-field>
<zero-text-field
  :value=${sync(x=> x.price)}
>
  Price
</zero-text-field>
<span>Instrument</span>
<zero-select
  :value=${sync(x=> x.instrument)}
>
</zero-select>
<span>Side</span>
<zero-select
  :value=${sync(x=> x.side)}
>
</zero-select>
```

You can now refresh your application; it should look something like this:

![](/img/position-form-2023_1.png)

:::note
The data in your grids may vary from the data in the example. You may also only see one grid or none at all, depending on whether you replaced or appended the xml before this.
:::

## Adding selection options
You probably realise that we don't have any options in our select components, so let's fix that now.

To enter a new trade, we want the user to be able to select:
- side (buy or sell)
- the instrument to be traded

We will start with side, as it only has two static options: BUY and SELL. We just need to add those two options inside the select tag:

```html title='home.template.ts'
<zero-select :value=${sync(x=> x.side)}>
    <zero-option>BUY</zero-option>
    <zero-option>SELL</zero-option>
</zero-select>
```

To enable the user to select the instrument, it's more complicated, because a list of options needs to be fetched from the API.

We will do that in [connectedCallback](https://www.fast.design/docs/fast-element/defining-elements#the-element-lifecycle), which happens when an element is inserted into the DOM.
First, declare `tradeInstruments`. This will be used in the template later.

To get the data from the API, inject:
```typescript title='home.ts'
@observable tradeInstruments: Array<{value: string, label: string}>;
@Connect connect: Connect;
public async connectedCallback() {
    super.connectedCallback();

    const tradeInstrumentsRequest = await this.connect.request('INSTRUMENT');
    this.tradeInstruments = tradeInstrumentsRequest.REPLY?.map(instrument => ({value: instrument.INSTRUMENT_ID, label: instrument.INSTRUMENT_ID}));
    this.instrument = this.tradeInstruments[0].value;
}
```

Once we have the data with the list of instruments, we can make use of it in the template file.
To dynamically include a list of instruments, use the [repeat](https://www.fast.design/docs/fast-element/using-directives#the-repeat-directive) directive and iterate through the items.

```typescript {2-4} title='home.template.ts'
<zero-select :value=${sync(x=> x.instrument)}>
  ${repeat(x => x.tradeInstruments, html`
    <zero-option value=${x => x.value}>${x => x.label}</zero-option>
  `)}
</zero-select>
```

## Enabling the user to insert
Now we have the data that can be selected, we need the user to be able to submit the trade:

Create a simple button with a click event handler:
```html title='home.template.ts'
<zero-button @click=${x=> x.insertTrade()}>Add Trade</zero-button>
```

Then create a new API call to insert the trade (this replaces the `insertTrade()` function you previous declared):
```typescript title='home.ts'
public async insertTrade() {
  const insertTradeRequest = await this.connect.commitEvent('EVENT_TRADE_INSERT', {
    DETAILS: {
      COUNTERPARTY_ID: 'GENESIS',
      INSTRUMENT_ID: this.instrument,
      QUANTITY: this.quantity,
      PRICE: this.price,
      SIDE: this.side,
      TRADE_DATETIME: Date.now(),
    },
    IGNORE_WARNINGS: true,
    VALIDATE: false,
  });
}
```
 Let's add another data grid at the bottom of the page to show the trade view `ALL_TRADES`:

```html {2,17-23}title='home.template.ts'
<div class="column-split-layout">
	<div class="row-split-layout">
		<zero-grid-pro persist-column-state-key="position-grid-settings">
			<grid-pro-genesis-datasource
				resource-name="ALL_POSITIONS"
				order-by="INSTRUMENT_ID"
			></grid-pro-genesis-datasource>
			${repeat(
				() => positionColumnDefs,
				html`
					<grid-pro-column :definition="${(x) => x}"></grid-pro-column>
				`
			)}
			<grid-pro-column :definition="${(x) => x.singlePositionActionColDef}"></grid-pro-column>
		</zero-grid-pro>

		<zero-grid-pro>
			<grid-pro-genesis-datasource
				resource-name="ALL_TRADES"
				order-by="INSTRUMENT_ID"
			></grid-pro-genesis-datasource>
		</zero-grid-pro>
	</div>

	<zero-text-field :value=${sync((x) => x.quantity)}>Quantity</zero-text-field>
	<zero-text-field :value=${sync((x) => x.price)} type="number">Price</zero-text-field>
	<span>Instrument</span>
	<zero-select :value=${sync((x) => x.instrument)}>
		${repeat(
			(x) => x.tradeInstruments,
			html`
				<zero-option value=${(x) => x.value}>${(x) => x.label}</zero-option>
			`
		)}
	</zero-select>
	<span>Side</span>
	<zero-select :value=${sync((x) => x.side)}>
		<zero-option>BUY</zero-option>
		<zero-option>SELL</zero-option>
	</zero-select>
	<zero-button @click=${(x) => x.insertTrade()}>Add Trade</zero-button>
</div>
```
Now if everything has worked, you can go to your browser, insert the data for a new trade, and then click the button. The new trade displays in the data grid of the trade view `ALL_TRADES` in the top right of the page.

![](/img/finished-trade-view.png)

## Conclusion
You can use the [positions app tutorial repo](https://github.com/genesiscommunitysuccess/positions-app-tutorial/tree/Complete_positions_app/client/web/src/routes/home) as a reference point for the forms.
