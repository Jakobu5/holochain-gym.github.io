# Concepts >> Dna and Zomes ||40

```js script
import "@rocket/launch/inline-notification/inline-notification.js";
import { html } from "lit-html";
import {
  HolochainPlaygroundContainer,
  EntryContents,
  EntryGraph,
  CallZomeFns,
  SelectActiveDna,
  SourceChain,
  DhtCells,
  selectCell
} from "@holochain-playground/elements";
import {
  GetStrategy,
  NetworkRequestType,
  WorkflowType,
  hash,
  HashType,
} from "@holochain-playground/core";
import { DnaCode, demoHapp } from "@holochain-playground/blockly";

customElements.define(
  "holochain-playground-container",
  HolochainPlaygroundContainer
);
customElements.define("entry-graph", EntryGraph);
customElements.define("entry-contents", EntryContents);
customElements.define("call-zome-fns", CallZomeFns);
customElements.define("source-chain", SourceChain);
customElements.define("dht-cells", DhtCells);
customElements.define("dna-code", DnaCode);
customElements.define("select-active-dna", SelectActiveDna);
```

**TLDR: each DNA creates its own DHT network, whose ID is the hash of its source code.**

<inline-notification type="tip" title="Useful reads">
<ul>
<li><a href="https://developer.holochain.org/concepts/2_application_architecture/">Core Concepts: Application Architecture</a></li>
<li><a href="https://en.wikipedia.org/wiki/Modularity">Modularity</a></li>
</ul>
</inline-notification>

# Dna

`Dnas` are one of the most important building blocks in Holochain. Simply put, a `Dna` is the source-code for the game you are playing with your peers in Holochain.

And here is the twist: in Holochain, **every Dna creates its own peer-to-peer DHT network**. This makes it so that **you are agreeing to the rules of the game before actually joining the network** and beginning to play with peers. This is going to be key for validation rules.

One of the most important characteristics of a `Dna` is that **it's identified by the hash of the source-code**. And that makes it so that, if you change something from that code and install that `Dna` again, you are literally creating a new network.

## Try it!

1. Edit the current `Dna` by clicking `Edit` in the `Dna Code` block.
2. Change something: it can be as simple as renaming a function, or removing a block.
3. Click `Compile`!
   - By this small change, you have changed the source code for the `Dna`.
   - This, at its turn, changes the hash of that `Dna`, so it's a new `Dna`.
   - By installing it in your node, you are creating a new Holochain DHT network.
   - But! You are the only one playing that game, so you are in a network of only one node.
4. Try to start playing with other people: select the initial `Dna` in the `Select Active Dna` block.
5. Select another agent.
6. Edit the `Dna`, with only the exact same modifications that you did in step 2.
7. Click `Compile`!
   - You should now see a network of 2 nodes! 
   - What has happened is that you both made the same modifications to the code, and as the ID of the `Dna` is the hash of the source code, that means that you are playing the same game, and hence you are in the same network.

```js story
export const Simple = () => {
  return html`
    <holochain-playground-container
      .numberOfSimulatedConductors=${4}
      .simulatedHapp=${demoHapp}
      @ready=${(e) => {
          const conductor = e.detail.conductors[0];

        const cellId = conductor.getAllCells()[0].cellId;

          console.log(e.target, cellId)
        e.target.activeAgentPubKey = cellId[1];
      }}
    >
      <div
        style="display: flex; flex-direction: row; margin-bottom: 20px; height: 600px;"
      >
        <dna-code
          style="flex: 1; margin-right: 20px;"
          @dna-compiled=${async (e) => {
            const context = e.target;
            const activeAgent = context.activeAgentPubKey;
            const cell = selectCell(
              context.activeDna,
              activeAgent,
              context.conductors
            );

            await cell.conductor.installHapp(
              {
                name: `blockly-happ${Date.now()}`,
                description: "",
                slots: {
                  default: {
                    deferred: false,
                    dna: e.detail.dna,
                  },
                },
              },
              {}
            );

            const dnaHash = hash(e.detail.dna, HashType.DNA);

            context.updatePlayground({
              activeDna: dnaHash,
            });
          }}
        ></dna-code>
        <div style="display: flex; flex-direction: column; flex: 1;">
          <select-active-dna
            style="margin-bottom: 20px; flex-grow: 0;"
          ></select-active-dna>
          <dht-cells id="dht-cells" style="flex:1;" hide-filter hide-time-controller></dht-cells>
        </div>
      </div>
      <div
        style="display: flex; flex-direction: row; margin-bottom: 20px; height: 350px;"
      >
        <call-zome-fns
          id="call-zome"
          style="margin-right: 20px; flex: 1;"
          hide-agent-pub-key
        >
        </call-zome-fns>
        <entry-graph
          show-entry-contents
          .excludedEntryTypes=${["Agent"]}
          style="flex: 1;"
        >
        </entry-graph>
      </div>
    </holochain-playground-container>
  `;
};
```

You can also create some entries in one `Dna` and switch back and forth `Dnas` to realise that when you create a new network, none of the data is ported.

## Zomes

`Dnas` in Holochain are composed out of multiple `zomes`. `Zomes` are like modules: you can compose them together to aggregate its functionality.

These are examples for different zomes:

- Roles zome
- Blogging zome
- File storage zome
- Currency zome
- Calendar zome
- Content management zome
- Index/filtering zome

And from this list we could group different zomes together to make different happs:

- Meetup clone:
  - Calendar zome
  - Communities zome
  - Currency zome
  - Roles zome
- Wikipedia clone:
  - Roles zome
  - Content management zome 
  - Index/filtering zome
- Twitter clone:
  - Blogging zome
  - File storage zome

You can start to see how zomes can add a lot of composability to Holochain's functionalities.

### Declarations

Each zome has to declare:

- Zome functions: these are the functions that can be executed from outside Holochain (the UI of a web-app, for example).
- Entry types: the kinds of entries that this zome is going to create in its zome functions.
- Validation rules: the rules by which the entry types of this zome are valid or not.

### Constraints

These are the constraints by which zomes are to play with each other:

- Each `zome` can see the entries committed by any other `zome`, both in the DHT and in the source chain.
- Each `zome` is responsible for validating the entries and links it creates.
- `Zomes` can call functions from each other.
- A `zome` can attach links to entries created from other `zomes`.
