# genyaml

## GEDCOM to YAML

### Why
[GEDCOM](https://en.wikipedia.org/wiki/GEDCOM) is now the de facto standard for genealogical data. However, beyond its limitations, it is full of shortcuts. Well, its purpose was to exchange genealogical data between software.

How about making it simple and **human-readable** so that anyone (also software) can write and read it.

### What
Genealogical data, as described by GEDCOM, contains between the others:

- individual records (INDI)
- family records (FAM)

linked together by references. Relations are well structured so let's try to keep them the most similar. But let's try to replace abbreviations of [Tags](http://wiki-en.genealogy.net/GEDCOM-Tags) **with real words** e.g.:

| GEDCOM TAG: | replaced with: | YAML type:           | notes: |
|-------------|----------------|----------------------|--------|
| BIRT        | birth          | dictionary           |        |
| DEAT        | death          | dictionary           |        |
| BURI        | burial         | dictionary           |        |
| MARR        | marriage       | dictionary           |        |
| PLAC        | place          | string               |        |
| TITL        | title          | string               |        |
| OCCU        | occupation     | string               |        |

_...etc_

### How
#### Relations
What if the family tree was presented in the **[YAML](https://en.wikipedia.org/wiki/YAML)** file. The simplest could look like this:

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
    parentsId: F1
families: # GEDCOM FAM records
  - id: F1
    partnerIds: 
      - I1
      - I2
    childIds: 
      - I3
```

| GEDCOM TAG:  | replaced with: | YAML type:           | notes:                                           |
|--------------|----------------|----------------------|--------------------------------------------------|
| FAMS         | familyIds      | list of strings      |                                                  |
| FAMC _{1}_   | parentsId      | string               |                                                  |
| FAMC _{n>1}_ | parents        | list of dictionaries | for different types like biological and adoption |
| HUSB, WIFE   | partnerIds     | list of strings      |                                                  |
| CHIL         | childIds       | list of strings      |                                                  |

#### Plurals for collections
In the GEDCOM, there is no easy way to recognize if a record is part of a collection or just a single item. There are of course some specifications out there for different versions. But you can't tell it from just looking at the file.

I think the good old convention of naming collections with a plural is a way to go. Of course a collection in YAML itself is also visible by `-` or `[]` e.g.:

| GEDCOM TAG: | replaced with: | YAML type:           | notes: |
|-------------|----------------|----------------------|--------|
| OBJE        | objects        | list of dictionaries |        |

#### Personal Name
[Personal Name](https://en.wikipedia.org/wiki/Personal_name) is hard thing to model. It's the set of names that the individual person is known. However, it highly depends on cultural context - synonyms, order of parts, their meaning. That's also why there is so many ideas for personal name in GEDCOM and why it still evolves there.

In GEDCOM specifications NAME can be a string / text value (with surname between slashes) or a list of such values or a list of objects. When in [5.5.1 version](https://www.tamurajones.net/GEDCOM/GEDCOM551.pdf) they can be distinguished by TYPE (one of aka | birth | immigrant | maiden | married | user defined), it is no more possible to do it in [5.5.5 version](https://www.tamurajones.net/GEDCOM/GEDCOM55Plus.pdf). However still both define NAME with number of possibilities (pieces) like NPFX, GIVN, NICK, SPFX, SURN and NSFX. 

That's why my proposal is simply to use:

```
legalFullName: Prince Rogers Nelson
normalShortName: Prince
otherKnownNames: [The Artist, Joey Coco, Jamie Starr]
```

| GEDCOM TAG: | replaced with:  | YAML type:           | notes: |
|-------------|-----------------|----------------------|--------|
| NAME        | legalFullName   | string               |        |
|             | normalShortName | string               |        |
|             | otherKnownNames | list of strings      |        |

## Store in folders
Working comfortably on a family tree is not only a human-readable file, but also a well-organized storage of data. We could use a specific folder structure for this. In each such folder, we could store a piece of data about a specific person or family. Simple tooling would make it possible to produce one file from all folders. The proposal is to reflect the relations in such folders per family tree:

- individuals/{person-name-unique}
- families/{family-id}

and place there all particular files with kind of _symlinks_ to be able to traverse the tree accordingly to relation structure.

## YAML to MARKDOWN
