# FitFindr — planning.md

> Complete this document before writing any implementation code.
> Your spec and agent diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Your planning.md will be reviewed as part of your submission.
> Update it before starting any stretch features.

---

## Tools

List every tool your agent will use. For each tool, fill in all four fields.
You must have at least 3 tools. The three required tools are listed — add any additional tools below them.

### Tool 1: search_listings

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->

This tool searches the listings dataset for items that match the user's requested item description, size, and maximum price. It then filters out listings that are too expensive and finds the most relevant matches.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `description` (str): Contains keywords describing the item that the user wants, such as "vintage graphic tee" or "black boots".
- `size` (str): It is the size of the item and it can also be "None" if no size is provided
- `max_price` (float): It contains the maximum amount that the user wants to spend and can be "None" if no specific budget is provided.

**What it returns:**
<!-- Describe the return value — what fields does a result contain? -->

A list of matching listing dictionaries containing fields such as the id, title, description, category, style_tages, size, condition, price, colors, brand, and platform

**What happens if it fails or returns nothing:**
<!-- What should the agent do if no listings match? -->

The tool returns an empty list. The model tells the user that no matching listings were found and suggests increasing the budget, changing the size filter, or using a better description.
---

### Tool 2: suggest_outfit

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->

It suggests a complete outfit using the selected items and prices based on the user's wardrobe. The goal is to create a coordinated look that matches the item's style.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `new_item` (dict): The selected item from search_listings
- `wardrobe` (dict): The user's wardrobe containing clothing items and other accessories. 

**What it returns:**
<!-- Describe the return value -->
It returns a string containing a complete outfit suggestion and styling advice. 

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the wardrobe is empty or no outfit can be suggested? -->

If the wardrobe is empty, the tool returns general styling advice for the selected item.

---

### Tool 3: create_fit_card

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->

This creates a short caption based on the selected item and outfit suggestion.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `outfit` (...): The outfit suggestion produced by suggest_outfit

**What it returns:**
<!-- Describe the return value -->
A short caption describing the outfit in a fun and shareable way.

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the outfit data is incomplete? -->

If the outfit input is missing or empty, the tool returns an error message explaining that a fit card could not be generated. 

---

### Additional Tools (if any)

<!-- Copy the block above for any tools beyond the required three -->

---

## Planning Loop

**How does your agent decide which tool to call next?**
<!-- Describe the logic your planning loop uses. What does it look at? What conditions change its behavior? How does it know when it's done? -->

The agent first extracts the item description, size, and maximum price based on the user's query and calls search_listings.

If no matches are found, the agent returns an error message and then stops. If there is a match, the selected item is passed to suggest_outfit along with the user's wardrobe.

The outfit suggestion is then passed to create_fit_card which generates a caption. The process then ends with either a fit card or an error message is returned. 

---

## State Management

**How does information from one tool get passed to the next?**
<!-- Describe how your agent stores and accesses state within a session. What data is tracked? How is it passed between tool calls? -->

The agent stores information in a session dictionary throughout the process. The selected item from the search_listings is saved and passed directly to suggest_outfit without having the user to enter it again. The outfit suggestion is then saved and passed into create_fit_card. Any errors are also stored in the session and returned to the user if the process stops early. 

---

## Error Handling

For each tool, describe the specific failure mode you're handling and what the agent does in response.

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query | The agent returns an error message and suggests to increase the budget, change the size filter, or using a broad description |
| suggest_outfit | Wardrobe is empty | This tool provides general styling advice and not specific outfit recommendations. |
| create_fit_card | Outfit input is missing or incomplete | This tool returns an error message explaining that a fit card could not be generated.|

---

## Architecture

<!-- Draw a diagram of your agent showing how the components connect:
     User input → Planning Loop → Tools (search_listings, suggest_outfit, create_fit_card)
                                                                          ↕
                                                                   State / Session
     Show what triggers each tool, how state flows between them, and where error paths branch off.
     ASCII art, a Mermaid diagram (https://mermaid.js.org/syntax/flowchart.html), or an embedded
     sketch are all fine. You'll share this diagram with an AI tool when asking it to implement
     the planning loop and each individual tool. -->

     User input → Planning Loop → Tools (search_listings - return error if no result, suggest_outfit, create_fit_card) → session["selected_item"] → suggest outfit → session["outfit_suggestion"] → create_fit_card → session["fit_card"] → Return final result to user

---

## AI Tool Plan

<!-- For each part of the implementation below, describe:
     - Which AI tool you plan to use (Claude, Copilot, ChatGPT, etc.)
     - What you'll give it as input (which sections of this planning.md, your agent diagram)
     - What you expect it to produce
     - How you'll verify the output matches your spec before moving on

     "I'll use AI to help me code" is not a plan.
     "I'll give Claude my Tool 1 spec (inputs, return value, failure mode) and ask it to implement
     search_listings() using load_listings() from the data loader — then test it against 3 queries
     before trusting it" is a plan. -->

**Milestone 3 — Individual tool implementations:**
I will give ChatGPT my Tool specifications based on function requirements, inputs, outputs, and failure conditions. After implementation, I tested the tools with different input values to verify that it produced expected results. 

**Milestone 4 — Planning loop and state management:**
I used ChatGPT to help implement the planning loop and state management. I provided the Planning Loop and the State Management sections along with tool specifications. After implementation, I tested a query to verify that the agent followed the correct workflow and handled errors

---

## A Complete Interaction (Step by Step)

Write out what a full user interaction looks like from start to finish — tool call by tool call. Use a specific example query.

**Example user query:** "I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would I style it?"

**Step 1:**
<!-- What does the agent do first? Which tool is called? With what input? -->

The agent extracts the description, size, and maximum price from the query and calls search_listings to find matching items

**Step 2:**
<!-- What happens next? What was returned from step 1? What tool is called now? -->

search_listings returns matching listings. The agent selects the top result and stores it as the selected item

**Step 3:**
<!-- Continue until the full interaction is complete -->

The selected item and the user's wardrobe are passed to suggest_outfit, which generates outfit recommendations using items from the wardrobe

**Final output to user:**
<!-- What does the user actually see at the end? -->

The outfit recommendation and selected item are passed to create_fit_card, which generates a short caption
