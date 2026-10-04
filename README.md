# World Hostility Graph

**Live demo:** https://hierarchical-edge-bundling.vercel.app/

An interactive hierarchical edge bundling diagram that shows which nations have been in militarized conflict with one another between 1816 and 2014. Countries are arranged around a circle and grouped by continent, conflicts are drawn as bundled lines between them, and a bar next to each country shows how many different countries it has clashed with. Hovering a country highlights its links and shows its statistics.

---

## Data Source

The data comes from the **Correlates of War (COW) Project**, a long-running academic research initiative founded in 1963 at the University of Michigan by J. David Singer. The project collects, maintains and publishes quantitative data sets on international relations and conflict, and its data is among the most widely used in political science and peace and conflict research.

This visualization uses the **Militarized Interstate Disputes (MID)** data set (https://correlatesofwar.org/data-sets/mids/). It records every instance in which one state threatened, displayed or used military force against another state between **1816 and 2014**. The COW Project defines a militarized interstate dispute as a historical case of conflict in which the threat, display or use of military force short of war by one state is explicitly directed at the government, officials, forces, property or territory of another state. Disputes range in intensity from threats to use force up to actual combat, and each dispute is assigned a hostility level, where the highest level (5) marks a war.

In practice the data provides, for each dispute:

- the states involved and the side each state was on,
- the start and end dates of the dispute and of each state's participation,
- the hostility level reached (threat, display, use of force, war),
- the highest action taken, the outcome and the number of fatalities.

In total, at least **4,726 conflicts** are recorded for this period. Many of them involved the use of military force, and some escalated into war.

**Citation:** Palmer, Glenn, Roseanne W. McManus, Vito D'Orazio, Michael R. Kenwick, Mikaela Karstens, Chase Bloch, Nick Dietrich, Kayla Kahn, Kellan Ritter, Michael J. Soules. 2020. "The MID5 Dataset, 2011–2014: Procedures, Coding Rules, and Description." *Conflict Management and Peace Science*, 39(4): 470–482.

---

## Combining the Data Source with Continent Information

The MID data identifies countries only by their COW country code and abbreviation; it contains no geographic grouping. To be able to arrange the countries by world region, I enhanced the existing data with **continent information**.

### Enrichment

Each COW country code was matched to a continent (Africa, Americas, Asia, Europe, Oceania). This continent attribute became the top level of the hierarchy that the edge bundling layout is built on:

```
World
 ├── Africa
 │    ├── Country
 │    └── ...
 ├── Americas
 ├── Asia
 ├── Europe
 └── Oceania
```

### Cleaning

Before the data could be used in the visualization, it was cleaned:

- **Historical states:** states that no longer exist or have changed (e.g. Prussia, the Soviet Union, Yugoslavia, Austria-Hungary) were checked and assigned a continent so that no country was left without a group.
- **Unmatched codes:** country codes without a continent match were identified and corrected manually.
- **Pairs of opponents:** the participant-level records were converted into pairs of countries on opposing sides of a dispute, since a link can only connect two countries.
- **Duplicates:** repeated pairs were merged, so that each pair of countries is connected by one link, regardless of how often they fought.
- **Self-links and empty entries:** records that would link a country to itself or had missing values were removed.
- **Aggregation:** for each country, the number of distinct opponents was counted to produce the value used for the bar length.

The result is a hierarchical JSON structure (continent → country) with a list of conflict links for every country.

---

## Data Display

The visualization is explained using Tamara Munzner's **Marks & Channels** framework. *Marks* are the basic geometric elements that represent items or links (points, lines, areas). *Channels* control the appearance of marks (position, color, length, etc.) and encode attributes of the data. Magnitude channels such as position and length suit ordered, quantitative data, while identity channels such as color hue suit categorical data.

| Data | Mark | Channel | Attribute type |
| --- | --- | --- | --- |
| Conflict between two countries | Line (link) | Connection | Relationship |
| Country's continent | Point (country label) | Position on the circle (grouping) | Categorical |
| Country's continent | Bar (area) | Color hue | Categorical |
| Number of opponents | Bar (area) | Length | Quantitative |

### Lines connect countries and symbolize conflict

Each conflict relationship is shown with a **line mark**. A line is a *link mark*: it does not stand for one item, but for a relationship between two items, which makes it the natural choice for showing that two countries were hostile to each other. A line is drawn between every pair of countries that have been on opposing sides of a militarized dispute.

The lines are drawn with **hierarchical edge bundling**: instead of running straight across the circle, they curve along the continent hierarchy toward the center. Links between the same regions are bundled together, which reduces visual clutter and reveals large-scale patterns, such as the many lines running between Europe and Asia, or the dense bundles going out from a few superpowers to every part of the world.

### Position of countries grouped as continents

Each country is a **point mark** (represented by its label) placed on the outer circle. The **position channel** is the most effective channel, and here it is used to encode the categorical continent attribute through **spatial grouping**: all countries of one continent sit next to one another in a shared segment of the circle. Because grouping by proximity is perceived immediately, the viewer can see at a glance which continent a country belongs to and whether a conflict stays within a continent (short lines inside one segment) or crosses continents (long lines across the circle).

### Bar color for continent information

Next to each country is a bar (an **area mark**). Its **color hue** encodes the continent. Hue is an *identity channel*: it tells the viewer *what* something is rather than *how much* of it there is, which makes it right for categorical data such as continents. Each continent gets one distinct color, which reinforces the grouping already given by position (redundant encoding) and makes it easier to tell where one continent's segment ends and the next one begins.

### Bar length for the amount of conflicts

The **length** of each bar encodes how many different countries a nation has been in conflict with. Length is a *magnitude channel* and is, after position, one of the most accurately perceived channels for quantitative values, so viewers can compare countries reliably: a long bar indicates a nation involved in hostilities with many other states, a short bar indicates only a few opponents. This makes it easy to spot that the involvement is very unequally divided and that a small number of international superpowers have clashed with a large number of countries, both near their borders and far away.

### Interaction

Hovering over a country highlights its links and dims the rest of the graph, while a tooltip shows the country's statistics. This makes it possible to explore individual countries in an otherwise very dense diagram.

---

## Data Credits

Data: [Correlates of War Project – Militarized Interstate Disputes (v5.0)](https://correlatesofwar.org/data-sets/mids/)
