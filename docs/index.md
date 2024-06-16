Kode24 lønn - 2024
==================

Tester [Observable framework](https://observablehq.com/framework/) for visualisering 
av [Kode24 sin lønnsdata for 2024](https://www.kode24.no/artikkel/her-er-lonnstallene-for-norske-utviklere-2024/81507953).

Kildekoden ligger på [Github](https://github.com/kimble/kode24-lonnsdata).


## Antagelser / tweaking

Jeg har sikkert gjort flere antagelser uten å være klar over det, men her er de antagelsene jeg bevisst har gjort.

1. Det er veldig få datapunkter for de med > 30 års erfaring. Alle som rapportert mer enn 30 års erfaring slår jeg sammen. 30 i grafene under er altså 30+.
2. Antar at folk har tolket "utdanning" forskjellig. Det ser ut som det varierer veldig mye om folk har telt med grunnskole eller ikke. Jeg klassifiserer de som har rapportert 3 års utdanning som bachelor og de med 5 som master.
3. Jeg har forkortet en del lange navn.

```js

const simplifyGender = (g) => g === "annet / ønsker ikke oppgi" ? "annet/ukjent" : g;
const simplifyTopic = (g) => g === "AI / maskinlæring" ? "ai/ml" : g === "embedded / IOT / maskinvare" ? "IOT" : g;

const workbook = await FileAttachment("data/lonnstall-2024.xlsx").xlsx();
const rawData = workbook.sheet("Form1", {range: "A:H", headers: true});

const uniqueSituations = [...new Set([...rawData.flatMap((d) => d["arbeidssituasjon"].split(", "))])].map((s) => s === "frilans / selvstendig næringsdrivende" ? "frilans" : s).map(s => s === "offentlig/kommunal sektor" ? "offentlig" : s)
const uniquePlaces = [...new Set([...rawData.flatMap((d) => d["arbeidssted"].split(", "))])];
const uniqueTopics = [...new Set([...rawData.flatMap((d) => d["fag"].split(", "))])].map(simplifyTopic);
const uniqueGenders = [...new Set([...rawData.flatMap((d) => d["kjønn"].split(", "))])].map(simplifyGender);

const data = rawData.map((d) => {
    const copy = {
        "gender": simplifyGender(d["kjønn"]),
        "topic": simplifyTopic(d["fag"]),
        "place": d["arbeidssted"],
        "education": d["utdanning"],
        "experience": d["erfaring"] < 30 ? d["erfaring"] : 30,
        "salary": d["lønn"],
        "bonus": d["bonus?"],
    };

    uniqueSituations.forEach((s) => copy[s] = d["arbeidssituasjon"].indexOf(s) > -1 ? "Ja" : "Nei");

    if (d["arbeidssituasjon"].indexOf("offentlig") > -1) {
        copy["sector"] = "public";
    } else if (d["arbeidssituasjon"].indexOf("privat") > -1) {
        copy["sector"] = "private";
    } else {
        copy["sector"] = "n/a";
    }
    
    if (copy["education"] === 3) {
        copy["grade"] = "bachelor";
    } else if (copy["education"] === 5) {
        copy["grade"] = "master";
    } else {
        copy["grade"] = "n/a"
    }

    return copy;
});
```

## Filtrer
Bruk disse til å se på forskjellige utsnitt av dataen. 

```js
const selectedPlaces = view(
    Inputs.checkbox(uniquePlaces, {label: "Arbeidssted", sort: true, unique: true, value: uniquePlaces})
);
```

```js
const selectedTopics = view(
    Inputs.checkbox(uniqueTopics, {label: "Fag", sort: true, unique: true, value: uniqueTopics})
);
```

```js
const selectedSituations = view(
    Inputs.checkbox(uniqueSituations, {label: "Situasjon", sort: true, unique: true, value: uniqueSituations})
);
```

```js
const selectedGenders = view(
    Inputs.checkbox(uniqueGenders, {label: "Kjønn", sort: true, unique: true, value: uniqueGenders})
);
```

## Data som matcher filter
Dette er de (masserte) radene som matcher det du har krysset av for ovenfor.  

```js
const filteredData = data.filter((d) => {
    return selectedPlaces.indexOf(d.place) > -1 && selectedTopics.indexOf(d.topic) > -1 && selectedSituations.some((s) => d[s] === "Ja") && selectedGenders.indexOf(d.gender) > -1
});
```

```js

const summarize = (data, predicate) => {
    const matchedRows = data.filter(predicate);
    const experienceExtent = d3.extent(matchedRows, (d) => d.experience);

    return Array(experienceExtent[1]+1).fill(0).map((_, i) => i).map((e) => {
        const salaries = matchedRows.filter(d => d.experience === e).map(d => d.salary);

        return {
            experience: e,
            salaries: salaries,
            mean: d3.mean(salaries),
            median: d3.median(salaries),
            p5: d3.quantile(salaries, 0.05),
            p95: d3.quantile(salaries, 0.95),
        }
    });
} 

```

```js
display(Inputs.table(filteredData, {
    
}));
```



## Erfaring vs. rapportert lønn 


```js
const salarySummary = summarize(filteredData, (d) => true);

view(
    resize((w) => {
        return Plot.plot({
            width: w,
            marginLeft: 80,
            inset: 10,
            grid: true,
            color: {
              legend: true,
            },
            x: {label: "Års erfaring →"},
            y: {label: "↑ Lønn"},
            marks: [
                Plot.ruleY([0]),
                Plot.areaY(salarySummary, { x: "experience", y1: "p5", y2: "p95", fill: "lightgray", "curve": "natural" }),
                Plot.line(salarySummary, {x: "experience", y: "median", curve: "natural", strokeDasharray: "3", stroke: "gray", opacity: 0.3}),
                Plot.dot(filteredData, {x: "experience", y: "salary", opacity: 0.7})
            ]
        })
    })
);
```

Etter ca. 10 år ser det ut som de fleste kan gi opp tanken på å gå særlig opp i lønn.

Legg merke til det smale spredningen i lønn rapportert av de med 22 års erfaring. Kan det hende at lønna til de som gikk ut av
skolen under finanskræsjet knyttet til [Stock market downturn of 2002](https://en.wikipedia.org/wiki/Stock_market_downturn_of_2002) 
**fortsatt** er påvirket av dette? De som gikk ut i jobb noen år før og etter (spesielt før...) bobla sprakk rapporterer betydelig 
høyere lønn i dag godt over tjue år senere.

Dersom du bruker filter på toppen av siden til å kun vise "ledelse/administrativt" ser vi at det ikke
er noen med 22 års erfaring i datasettet som har rapportert lønn. Dårlig jobbmarked for mellomledere etter 
at lufta gikk ut av bobla?

Det kan også se ut som vi kan se spor av [2015-2016 stock market selloff](https://en.wikipedia.org/wiki/2015%E2%80%932016_stock_market_selloff)? 
De som har rapportert 9 års arbeidserfaring gikk ut av skolen på den tiden. 

## Spredning i rapportert lønn etter arbeidserfaring 

```js
view(
    resize((w) => {
        return Plot.plot({
            fy: {
                grid: true,
                reverse: false,
                label: "Års erfaring"
            },
            x: {label: "Lønn →"},
            marks: [
                Plot.boxX(filteredData, {x: "salary", fy: "experience"})
            ]
        })
    })
);
```

Det høres rett ut at spredningen i rapportert lønn er minst i starten av karrieren. Er vel få arbeidsgivere som har
lyst til å bla opp store penger før de har fått erfare hvor produktiv en person er.


## Bachelor vs. master

**Obs!** Her var det en del rariteter i datasettet, se antagelser i toppen for å se hvilke antagelser jeg har tatt. 

```js
const bachelorSummary = summarize(filteredData, (d) => d.grade === "bachelor").map((d) => ({...d, grade: "bachelor"}));
const masterSummary = summarize(filteredData, (d) => d.grade === "master").map((d) => ({...d, grade: "master"}));
const bothSummary = [...bachelorSummary, ...masterSummary];


view(
    resize((w) => {
        return Plot.plot({
            width: w,
            marginLeft: 80,
            inset: 10,
            grid: true,
            color: {
                legend: true,
            },
            x: {label: "Års erfaring →"},
            y: {label: "↑ Lønn"},
            marks: [
                Plot.ruleY([0]),
                Plot.areaY(bothSummary, { x: "experience", y1: "p5", y2: "p95", fill: "grade", "curve": "natural", opacity: 0.5 }),
                Plot.line(bothSummary, {x: "experience", y: "median", curve: "natural", strokeDasharray: "3", stroke: "grade"})
            ]
        })
    })
);
```

Som ventet ligger de med mastergrad litt over de med bachelorgrad.

Men hvor mange år må en med mastergrad jobbe før hen tar igjen en som "bare" har tatt bachelorgrad og kommet ut i jobbmarkedet to år tidligere? 

**Obs!** Jeg er verken flink med statistikk og økonomi eller sammenligningen nedenfor har en rekke svakheter. Grafen nedenfor
tar utgangspunkt i medianen til folk med bachelor/master og x antall års erfaring. Poenget er at man med bachelorgrad soper inn
ish en million før skatt mens en som tar mastergrad sitter på skolebenken. Som Rema 1000 sier, det er sluttsummen på kassalappen som teller.

```js
const masterAndBachelorExtent = d3.extent(bothSummary, (d) => d.experience);

const bachelorVsMasterExperienceData = Array(masterAndBachelorExtent[1]+1).fill(0).map((_, i) => i).map((e) => {
    return {
        experience: e,
        bachelorMedian: bothSummary.find((d) => d.experience === e && d.grade === "bachelor").median || bothSummary.find((d) => d.experience === e-1 && d.grade === "master").median, // Take previous if missing
        masterMedian: bothSummary.find((d) => d.experience === e && d.grade === "master").median || bothSummary.find((d) => d.experience === e-1 && d.grade === "master").median, // Take previous if missing
    }
});

const bachelorVsMaterCumulativeData = Array(masterAndBachelorExtent[1]+1).fill(0).map((_, i) => i).map((year) => {
    const cumulativeBachelor = d3.sum(bachelorVsMasterExperienceData.filter(d => d.experience <= year), (d) => d.bachelorMedian);
    const cumulativeMaster = year >= 2 ? d3.sum(bachelorVsMasterExperienceData.filter(d => d.experience <= year - 2), (d) => d.masterMedian) : 0;
    
    return {
        year: year,
        medianSalaryBachelor: bachelorVsMasterExperienceData.find(d => d.experience === year).bachelorMedian,
        cumulativeBachelor: cumulativeBachelor,
        medianSalaryMaster: year >= 2 ? bachelorVsMasterExperienceData.find(d => d.experience === year-2).masterMedian : 0,
        cumulativeMaster: cumulativeMaster,
    }
});


const plottableBachelorVsMaster = [
    ...bachelorVsMaterCumulativeData.map(d => ({
        grade: "bachelor",
        year: d.year,
        cumulative: d.cumulativeBachelor
    })),
    ...bachelorVsMaterCumulativeData.map(d => ({
        grade: "master",
        year: d.year,
        cumulative: d.cumulativeMaster
    })),
];

view(
    resize((w) => {
        return Plot.plot({
            width: w,
            marginLeft: 80,
            inset: 10,
            grid: true,
            color: {
                legend: true,
            },
            x: {label: "Års erfaring →"},
            y: {label: "↑ Lønn"},
            marks: [
                Plot.ruleY([0]),
                Plot.line(plottableBachelorVsMaster, {x: "year", y: "cumulative", stroke: "grade", curve: "natural"}),
            ]
        })
    })
);

view(Inputs.table(bachelorVsMaterCumulativeData));
```

Dersom man også hadde tatt hensyn til to år med ekstra studielån og to år med tapt verdistigning i boligmarkedet blir
forskjellen naturligvis enda større i favør av å nøye seg med en bachelorgrad   . 