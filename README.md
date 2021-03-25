# genyaml

## GEDCOM to yaml

### WHY
[GEDCOM](https://en.wikipedia.org/wiki/GEDCOM) is now the de facto standard for genealogical data. However, beyond its limitations, it is full of shortcuts. Well, its purpose was to exchange genealogical data between software.

How about making it simple and human-readable so that anyone (also software) can write and read it.

### WHAT
Genealogical data, as described by GEDCOM, contains between the others:

- individual records (INDI)
- family records (FAM)

linked together by references. Relations are well structured so let's try to keep them the most similar. But let's try to replace abbreviations of [Tags](http://wiki-en.genealogy.net/GEDCOM-Tags) with real words.

### HOW
What if the family tree was presented in the [YAML](https://en.wikipedia.org/wiki/YAML) file. The simplest could look like this:

```yaml
# Yes, YAML allows us to comment
individuals: # GEDCOM INDI records
  - id: I1
    familyIds: 
      - F1
  - id: I2
    familyIds: 
      - F1
  - id: I3
    parentsId:
      - F1
families: # GEDCOM FAM records
  - id: F1
    partnerIds: 
      - I1
      - I2
    childIds: 
      - I3
```

#### References
| GEDCOM TAG: | replaced with: | YAML type:           | notes: |
|-------------|----------------|----------------------|--------|
| FAMS        | familyIds      | list of strings      |        |
| FAMC        | parentsId      | string               |        |
| HUSB, WIFE  | partnerIds     | list of strings      |        |
| CHIL        | childIds       | list of strings      |        |
