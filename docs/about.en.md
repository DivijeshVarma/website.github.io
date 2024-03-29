---
title: Heading
---

###  Divi MarkdownGuide Reference

[Link](https://www.markdownguide.org/basic-syntax/#overview)

### Markdown Cheat Sheet

[Link](https://www.markdownguide.org/cheat-sheet/)

### Extended Syntax

[Link](https://www.markdownguide.org/extended-syntax/)

### MkDocs Markdown Support

[Link](https://www.markdownguide.org/tools/mkdocs/)


### Markdown Hacks

[Link](https://www.markdownguide.org/hacks/#overview)

Some of these words <ins>will be underlined</ins>.



Avatar

[![Avatar](https://img.youtube.com/vi/d9MyW72ELq0/0.jpg)](https://www.youtube.com/watch?v=d9MyW72ELq0)


### Material for MkDocs

Reference:

[Link](https://squidfunk.github.io/mkdocs-material/reference/)

Setup:

[Link](https://squidfunk.github.io/mkdocs-material/setup/changing-the-colors/)





Blockquotes:

> Dorothy followed her through many of the beautiful rooms in her castle.



**Bold**


> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.



Blockquotes with Other Elements:

> #### The quarterly results look great!
>
> - Revenue was off the chart.
> - Profits were higher than ever.
>
>  *Everything* is going according to **plan**.


---

Ordered List Best Practices:

1. First item
2. Second item

Unordered Lists:

- First item
- Second item
- Third item
    - Indented item
    - Indented item
- Fourth item



---

Admonitions:
 
 Options:

abstract:  info: tip: success: question: warning: failure: 
danger: bug: example: quote:

!!! danger "danger" 

    Each of the supported admonition types has a distinct icon


---

1. First item
    - Indented item
2. Second item

---

[Link](https://github.com/orgs/privacyguides/people) 

My favorite search engine is [Duck Duck Go](https://duckduckgo.com "The best search engine for privacy").


---

URLs and Email Addresses

To quickly turn a URL or email address into a link, enclose it in angle brackets.

<https://www.markdownguide.org>

<fake@example.com>

---

Formatting Links:

I love supporting the **[EFF](https://eff.org)**.

This is the *[Markdown Guide](https://www.markdownguide.org)*.

See the section on [`code`](#code).

---

:fontawesome-brands-fedora: fedora brand

---

<mark> mark color</mark>

## second heading

1. one
2. two


### third heading

* dots
#### fourth heading

*Italic*

"asset"


:fontawesome-brands-redhat:

:fontawesome-brands-redhat:{ .redhat }

:octicons-heart-fill-24:{ .heart }

[Subscribe to our newsletter](#){ .md-button }

[Subscribe to our newsletter](#){ .md-button .md-button--primary }

[Send :fontawesome-solid-paper-plane:](#){ .md-button }


```py 
import tensorflow as tf
```

```mysql
apt install htop
```

[]: # (Tables)

| Method      | Description                          |
| :---------: | :----------------------------------: |
| `GET`       | :material-check:     Fetch resource  |
| `PUT`       | :material-check-all: Update resource |
| `DELETE`    | :material-close:     Delete resource |

[]: # (Formatting)

Text can be {--deleted--} and replacement text {++added++}. This can also be
combined into {~~one~>a single~~} operation. {==Highlighting==} is also
possible {>>and comments can be added inline<<}.

{==

Formatting can also be applied to blocks by putting the opening and closing
tags on separate lines and adding new lines between the tags and the content.

==}

- ==This was marked==
- ^^This was inserted^^
- ~~This was deleted~~


[]: # (Adding keyboard keys)

++ctrl+alt+del++


[]: # (Image)
  
   ![Tux, the Linux mascot](https://mdg.imgix.net/assets/images/tux.png?auto=format&fit=clip&q=40&w=100)



[]: # (List, unordered)
- Nulla et rhoncus turpis. Mauris ultricies elementum leo. Duis efficitur
  accumsan nibh eu mattis. Vivamus tempus velit eros, porttitor placerat nibh
  lacinia sed. Aenean in finibus diam.

    * Duis mollis est eget nibh volutpat, fermentum aliquet dui mollis.
    * Nam vulputate tincidunt fringilla.
    * Nullam dignissim ultrices urna non auctor.

-------------------------------

[]: # (List, ordered)
1.  Vivamus id mi enim. Integer id turpis sapien. Ut condimentum lobortis
    sagittis. Aliquam purus tellus, faucibus eget urna at, iaculis venenatis
    nulla. Vivamus a pharetra leo.

    1.  Vivamus venenatis porttitor tortor sit amet rutrum. Pellentesque aliquet
        quam enim, eu volutpat urna rutrum a. Nam vehicula nunc mauris, a
        ultricies libero efficitur sed.

    2.  Morbi eget dapibus felis. Vivamus venenatis porttitor tortor sit amet
        rutrum. Pellentesque aliquet quam enim, eu volutpat urna rutrum a.

        1.  Mauris dictum mi lacus
        2.  Ut sit amet placerat ante
        3.  Suspendisse ac eros arcu

----------------------------------

[]: # (Definition list)
`Lorem ipsum dolor sit amet`

:   Sed sagittis eleifend rutrum. Donec vitae suscipit est. Nullam tempus
    tellus non sem sollicitudin, quis rutrum leo facilisis.

`Cras arcu libero`

:   Aliquam metus eros, pretium sed nulla venenatis, faucibus auctor ex. Proin
    ut eros sed sapien ullamcorper consequat. Nunc ligula ante.

    Duis mollis est eget nibh volutpat, fermentum aliquet dui mollis.
    Nam vulputate tincidunt fringilla.
    Nullam dignissim ultrices urna non auctor.

---------------------------------
[]: # (Task list)
- [x] Lorem ipsum dolor sit amet, consectetur adipiscing elit
- [ ] Vestibulum convallis sit amet nisi a tincidunt
    * [x] In hac habitasse platea dictumst
    * [x] In scelerisque nibh non dolor mollis congue sed et metus
    * [ ] Praesent sed risus massa
- [ ] Aenean pretium efficitur erat, donec pharetra, ligula non scelerisque

--------------

[]: # (Adding abbreviations)
The HTML specification is maintained by the W3C.

*[HTML]: Hyper Text Markup Language
*[W3C]: World Wide Web Consortium

