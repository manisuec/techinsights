---
background: /img/posts/04.jpg
date: '2019-10-12T00:00:00Z'
lastmod: 2024-09-05T00:00:30
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/React-Dark_trdwyz.jpg'
title: Image Grid using Material UI in React
tags: ['reactjs', 'material-ui', 'image grid', 'grid']
keywords: 'reactjs,material-ui,image grid,grid,javascript'
categories: ['Reactjs']
url: 'reactjs/react-materialui-imagegrid'
aliases:
    - /post/2019-10-12-imagegrid-materialui-react/
---

A simple example of a scrollable image grid using Material UI in ReactJS. We had a requirement for a scrollable image grid which will load images lazily. We started with the [Grid List](https://material-ui.com/components/grid-list/) example provided in Material UI.

We added a few more capabilities

- Load images lazily
- On clicking the tile, display image dialog
- Download option on image dialog

I would request someone to update the example with below capabilities:

- Carousel
- Rotate images
- Zoom in/out

Few screenshots of the image grid

1. Image Grid ![Image Grid](https://res.cloudinary.com/dkiurfsjm/image/upload/v1725520524/ss_0_yawyfq.png)
2. Expaned Image ![Expanded Image](https://res.cloudinary.com/dkiurfsjm/image/upload/v1725520524/ss_1_vk6k8h.png)

We also used [React Infinite Scroller](https://github.com/CassetteRocks/react-infinite-scroller) which is a simple, lightweight infinite scroll package that supports both window and scrollable elements.

The image metadata array provided is hardcoded in the example code. This can be passed to the imagegrid component as a prop and fetched from your application's api endpoint. For example; we store the metadata in our database and actual images in different resolutions are stored in Amazon S3.

Complete code is checked in at [GitHub](https://github.com/manisuec/Sandbox).

You can play around with at [CodeSandbox](https://codesandbox.io/embed/imagegrid-lepk2)

