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
#### Relations
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

| GEDCOM TAG: | replaced with: | YAML type:           | notes: |
|-------------|----------------|----------------------|--------|
| FAMS        | familyIds      | list of strings      |        |
| FAMC        | parentsId      | string               |        |
| HUSB, WIFE  | partnerIds     | list of strings      |        |
| CHIL        | childIds       | list of strings      |        |

#### Further considerations
##### Personal Name
[Personal Name](https://en.wikipedia.org/wiki/Personal_name) is hard thing to model. It's the set of names that the individual person is known. However, it highly depends on cultural context - synonyms, order of parts, their meaning. That's also why there is so many ideas for personal name in GEDCOM and why it still evolves there.

In GEDCOM specifications NAME can be a string / text value (with surname between slashes) or a list of such values or a list of objects. When in [5.5.1 version](https://www.tamurajones.net/GEDCOM/GEDCOM551.pdf) they can be distinguished by TYPE (one of aka | birth | immigrant | maiden | married | user defined), it is no more possible to do it in [5.5.5 version](https://www.tamurajones.net/GEDCOM/GEDCOM55Plus.pdf). However still both define NAME with number of possibilities (pieces) like NPFX, GIVN, NICK, SPFX, SURN and NSFX. 

That's why my proposal is simply to use:

```
legalFullName: Prince Rogers Nelson
normalShortName: Prince
otherKnownNames: [The Artist, Joey Coco, Jamie Starr]
```

##### Plurals for collections
In the GEDCOM, there is no easy way to recognize if a record is part of a collection or just a single item. There are of course some specifications out there for different versions. But you can't tell it from just looking at the file.

I think the good old convention of naming collections with a plural is a way to go. Of course a collection in YAML itself is also visible by `-` or `[]`.
