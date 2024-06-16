Kode24 lønn - 2024
==================

```js

const simplifyGender = (g) => g === "annet / ønsker ikke oppgi" ? "annet/ukjent" : g;
const simplifyTopic = (g) => g === "AI / maskinlæring" ? "ai/ml" : g;

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
        "experience": d["erfaring"],
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

    return copy;
});
```

## Filtrer på arbeidssted

```js
const selectedPlaces = view(
    Inputs.checkbox(uniquePlaces, {sort: true, unique: true, value: uniquePlaces})
);
```

## Filtrer på fag

```js
const selectedTopics = view(
    Inputs.checkbox(uniqueTopics, {sort: true, unique: true, value: uniqueTopics})
);
```


Rader som matcher filter
------------------------

```js
const filteredData = data.filter((d) => {
    return selectedPlaces.indexOf(d.place) > -1 && selectedTopics.indexOf(d.topic) > -1
});
```

```js
display(Inputs.table(filteredData, {

}));
```



```js

```