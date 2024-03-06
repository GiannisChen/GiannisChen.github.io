---
icon: fa-solid fa-book
order: 4
---

> **Hello!**🎉 <br />
>  
> 📕 This is my bookshelf, which contains my reading list and notes.
{: .prompt-tip}

## Reading List

|       |     Title     |    Author    |   Tags   |      Description      |
| :---: | :-----------: | :----------: | :------: | :-------------------: |
|       |    空城计     |    呼延云    | Whodunit | recommended by 怪异君 |
|       | Queen国名系列 | Ellery Queen | Whodunit | recommended by 怪异君 |


## Reading Notes

- `Suspense`（悬疑）👻
- `Whodunit`（推理）🕵️‍
- `Unknown`（待定）❓

{% assign sorted = site.notes | sort: 'title' %}           
{% for note in sorted %}{% unless note.omit == true %}| [《{{ note.title }}》 - &nbsp; {{note.subtitle}}]({{ note.url | relative_url }})  | {{ note.author }} | {{note.tag}} | 
{% endunless %}{% endfor %}