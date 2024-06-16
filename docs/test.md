Test
====



```js

const simplifyGender = (g) => g === "annet / ønsker ikke oppgi" ? "annet/ukjent" : g;
const simplifyTopic = (g) => g === "AI / maskinlæring" ? "ai/ml" : g;

const workbook = await FileAttachment("data/lonnstall-2024.xlsx").xlsx();
const rawData = workbook.sheet("Form1", {range: "A:H", headers: true});

const situations = [...new Set([...rawData.flatMap((d) => d["arbeidssituasjon"].split(", "))])].map((s) => s === "frilans / selvstendig næringsdrivende" ? "frilans" : s).map(s => s === "offentlig/kommunal sektor" ? "offentlig" : s)
const places = [...new Set([...rawData.flatMap((d) => d["arbeidssted"].split(", "))])];
const topic = [...new Set([...rawData.flatMap((d) => d["fag"].split(", "))])].map(simplifyTopic);
const gender = [...new Set([...rawData.flatMap((d) => d["kjønn"].split(", "))])].map(simplifyGender);

const data = rawData.map((d) => {
    const copy = {
        "gender": simplifyGender(d["kjønn"]),
        "topic": simplifyTopic(d["fag"]),
        "place" : d["arbeidssted"],
        "education" : d["utdanning"],
        "experience" : d["erfaring"],
        "salary" : d["lønn"],
        "bonus" : d["bonus?"],
    };
    
    situations.forEach((s) => copy[s] = d["arbeidssituasjon"].indexOf(s) > -1 ? "Ja" : "Nei");
    return copy;
});
```

```js
display(Inputs.table(data, {

}));
```



```js

```