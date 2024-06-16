Test
====



```js
const workbook = await FileAttachment("data/lonnstall-2024.xlsx").xlsx();
const rawData = workbook.sheet("Form1", {range: "A:H", headers: true});
```


```js
display(rawData);
```

```js
display(Inputs.table(rawData, {
    columns: [
        "kjønn",
        "utdanning",
        "erfaring",
        "arbeidssted",
        "arbeidssituasjon",
        "fag",
        "lønn",
        "bonus?",
    ]
}));
```



```js
display(PieChart)
```