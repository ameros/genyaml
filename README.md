# genyaml

## GEDCOM to YAML

### Why
[GEDCOM](https://en.wikipedia.org/wiki/GEDCOM) is now the de facto standard for genealogical data. However, beyond its limitations, it is full of shortcuts. Well, its purpose was to exchange genealogical data between software.

How about making it simple and **human-readable** so that anyone (also software) can write and read it.

### What
Genealogical data, as described by GEDCOM, contains between the others:

- individual records (INDI)
- family records (FAM)

linked together by references.
![](https://docs.google.com/drawings/d/e/2PACX-1vRe0fpGo0MYDqlbQT1aQqYZOCNLF4UK9bWG6jLMIOQG3uW9v_wvfLAXPUD9HcPh4Tq4SgyWM33t4wPH/pub?w=958&h=339)
Relations are well structured so let's try to keep them the most similar. 

### How
Let's start with replacing abbreviations of [Tags](http://wiki-en.genealogy.net/GEDCOM-Tags) **with real words** e.g.:

| GEDCOM TAG: | replaced with: | YAML type:           | notes: |
|-------------|----------------|----------------------|--------|
| BIRT        | birth          | dictionary           |        |
| DEAT        | death          | dictionary           |        |
| BURI        | burial         | dictionary           |        |
| MARR        | marriage       | dictionary           |        |
| PLAC        | place          | string               |        |

_...etc_
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

I think the good old convention of naming collections with a plural is a way to go _(of course a collection in YAML itself is also visible by `-` or `[]`)_ e.g.:

| GEDCOM TAG: | replaced with: | YAML type:           | notes: |
|-------------|----------------|----------------------|--------|
| OBJE        | objects        | list of dictionaries |        |
| TITL        | titles         | list of strings      |        |
| OCCU        | occupations    | list of strings      |        |
| EDU         | educations     | list of strings      |        |

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

By the way, I really think that a maiden name could be written as `legalFullName` but under `birth` like:

```yaml
birth:
  date: 28 JUL 1929
  place: Southampton, New York, U.S.
  legalFullName: Jacqueline Lee Bouvier
```

## Store in folders
Working comfortably on a family tree is not only a human-readable file, but also a well-organized storage of data. We could use a specific folder structure for this. In each such folder, we could store a piece of data about a specific person or family. Simple tooling would make it possible to produce one file from all folders. The proposal is to reflect the relations in such folders per family tree:

- `individuals/{person-name-unique}/`
- `families/{family-id}/`

and place there all particular files with kind of _symlinks_ to be able to traverse the tree (folders) accordingly to relation structure.

## YAML to MARKDOWN
While we're here, why not use GitHub with Markdown files to conveniently navigate the tree 🤔 From the [About READMEs](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/about-readmes)

> GitHub will recognize and automatically surface your README to repository visitors.

So, if we could make simple transformation of YAML file to the README.md in each folder then we could also make use of Markdown links to refer relative folder READMEs.

Example YAML file:

```yaml
# John Fitzgerald Kennedy
id: I1
legalFullName: John Fitzgerald Kennedy
normalShortName: John F. Kennedy
otherKnownNames: [JFK, John Kennedy]
titles: [35th President of the United States, Senator, Congressman]
occupations: [politician]
educations: [Harvard University]
birth:
  date: 27 MAY 1917
  place: Brookline, Massachusetts, U.S.
death:
  date: 22 NOV 1963
  place: Dallas, Texas, U.S.
  cause: assassination
burial:
  place: Arlington National Cemetery
objects:
  - file: https://upload.wikimedia.org/wikipedia/commons/thumb/c/c3/John_F._Kennedy%2C_White_House_color_photo_portrait.jpg/370px-John_F._Kennedy%2C_White_House_color_photo_portrait.jpg
    title: John F. Kennedy, photograph in the Oval Office by Cecil Stoughton, White House; Public Domain
    format: jpg
familyIds:
  # with Jacqueline Lee Bouvier
  - F1
```

could be easily transformed to such simple Markdown:

```markdown
# John Fitzgerald Kennedy
- id: I1
- legalFullName: John Fitzgerald Kennedy
- normalShortName: John F. Kennedy
- otherKnownNames: JFK, John Kennedy
- titles: 35th President of the United States, Senator, Congressman
- occupations: politician
- educations: Harvard University
- birth:
  - date: 27 MAY 1917
  - place: Brookline, Massachusetts, U.S.
- death:
  - date: 22 NOV 1963
  - place: Dallas, Texas, U.S.
  - cause: assassination
- burial:
  - place: Arlington National Cemetery
- objects:
  - file: ![](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c3/John_F._Kennedy%2C_White_House_color_photo_portrait.jpg/370px-John_F._Kennedy%2C_White_House_color_photo_portrait.jpg)
    - title: John F. Kennedy, photograph in the Oval Office by Cecil Stoughton, White House; Public Domain
    - format: jpg
- familyIds:
  - F1 ([with Jacqueline Lee Bouvier](../../families/F1))
```

where the rules of transformation are easily visible and clear...

And _**voilà**_, there we got it - **human-readable** and **well-organized** files representing family tree 🎄

Please check the example starting from [John Fitzgerald Kennedy](https://github.com/ameros/genyaml/tree/main/examples/kennedy/individuals/John-Fitzgerald-Kennedy). 

There are also some templates already prepared for:
- [individuals](templates/individuals/)
- [families](templates/families/)

---

Now, all we need is just simple toolset
- to convert from YAML to Markdown
- from all YAML files to single one
- and, maybe still if someone interested, from that one YAML back to GEDCOM 🤔

## TBD

 Of course, there are many more in the GEDCOM standard (TAGS, type of RECORDS) to which the above proposal requires adjustment. Still, the base seems to be solid, and this adjustment should be easy.

## Google Sheets Template
Here comes something additional. I've just created [Google Family Tree Template](https://docs.google.com/spreadsheets/d/1Mgh86Fz51c6W2l5oUPLssexaeVqemhB1xN-zIzIToF4/edit?usp=sharing) that allows to collect GEDCOM-style data as linked sheets. You can walk it through by clicking links to parents, marriages and children. Such complete view on family tree members might be sometimes helpful and convenient.
 
## Feedback
Any feedback is appreciated  https://github.com/ameros/genyaml/discussions

_____
This proposal is listed @ https://www.cyndislist.com/gedcom/gedcom-software/

http://www.cyndislist.com/create-a-link-to-cyndis-list/
