# Recipe

🚨 **Important Update**: We have moved all our recipes to the official [Sectors API Docs](https://github.com/supertypeai/sectors_api_docs). You can now find the most up-to-date and enhanced versions of these recipes [there](https://github.com/supertypeai/sectors_api_docs/recipes). Previously, our recipes were available in this `recipe` directory in Markdown (`.md`) format. In our new documentation, they have been upgraded to MDX (`.mdx`) format to provide a more interactive and rich content experience. 

Each recipe is a standalone document that contains a brief introduction, a list of prerequisites, a step-by-step guide to follow, and some additional notes or tips that the author finds useful.

The recipes are designed to be easy to follow and can be used as a reference guide when working with the Sectors API. Be mindful and courteous when asking for help on the [Supertype Collective's Discord](https://discord.gg/TAnZMmNS4X) as most of these guides are contributed by the community and maintained by the community.

# Community Contributor
If you wish to be a community contributor, please drop @samuel a message [on our Discord](https://discord.gg/TAnZMmNS4X) so we can give you the necessary access to contribute to this repository. By becoming a contributor, you acknowledge that your work will be licensed under the MIT License and will be made available to the public. You will get full credit for your work and will be able to showcase your work to the broader audience. 

## Benefits
Being a community contributor, you'll be eligible for the following benefits:
- A free upgrade to the [Standard tier on your Sectors account](https://sectors.app/pricing) for 6 months, renewable for another 6 months if you contribute more than 5 recipes  
- Cash reward of IDR500,000 if you have more than 5 recipes merged into the repository
- Direct access to the developers and the team building Sectors at Supertype, where you can ask questions, get help, and provide feedback. You will also be able to participate in the roadmap discussions and feature requests, playing a crucial role in bridging the gap between the community (and Sector users) and the developers    
- A chance to be featured on the [Supertype Collective](https://collective.supertype.ai) platform, our [Supertype Blog](https://supertype.ai/notes) or Sectors community newsletter  

If we use any of your recipe in our official documentation, you will be credited and will be given a shoutout in our community channels.

## Contributing to the Cookbook
All recipes are to be contributed in the form of a markdown file with images and code snippets. The file should be named in the format `xx_title.md` where `xx` is the sequence number of the recipe and `title` is the title of the recipe. You should **submit a pull request** to this repository with your recipe, and it will be reviewed by the maintainers. If you are [part of the Supertype Fellowship program](https://fellowship.supertype.ai), a successful merge will earn you the corresponding points in the program.

You can also contribute by suggesting improvements to the existing recipes, fixing typos, or updating the content to reflect the latest changes in the Sectors API.

When writing your recipe, include a `yaml` front matter with the following fields:
```yaml
---
title: "Title of the Recipe"
author: "Your Name"
author_link: "Your GitHub Profile or Personal Website"
date: "YYYY-MM-DD"
language: "Python"
thumbnail: "URL to the thumbnail image if any"
---
```

You are encouraged to use Python, R, JavaScript or any other programming language that you are comfortable with. The code snippets should be well-documented and should be easy to follow. You can include images (by adding them to the `/image` sub-directory, and then referencing them), charts, or any other visual elements to make your recipe more engaging. Include code chunks with proper formatting ("code fences"). 

A typical recipe contains the following sections:
- Introduction
- Prerequisites (libraries, tools, etc.)
- Importing the data from Sectors API 
- Performing data wrangling using `pandas`, `dplyr`, or any other data manipulation process fit for the language
- Aggregation or visualization using any charting library
- Conclusion

### Examples 
You will find existing examples in this repository that you can use as a reference when writing your recipe.

Your work should be fully markdown compliant. A tip is to use the Markdown Preview or a markdown preview extension in VSCode so you can visualize the output. If you use Jupyter or RStudio, compile it down to markdown. Show all code chunks in your work including:
- all `import` statements for any libraries used
- provide simple installation instruction for any libraries used
- explain the process to obtain an API access key from Sectors and use it in your work
- explain the preprocessing and data-wrangling

A very good standard to follow for cookbook is:
- https://r-graphics.org/
- https://bbc.github.io/rcookbook/

If you are a [Supertype Fellowship member](https://fellowship.supertype.ai) with an assigned mentor (Fellowship+), you should consult with your mentor and receive the necessary guidance as you develop your work.
