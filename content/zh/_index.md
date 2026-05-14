---
title: ''
summary: ''
date: 2022-10-24
type: landing

design:
  spacing: '5rem'

sections:
  - block: resume-biography
    content:
      username: me
      text: ''
      # button:
      #   text: 下载简历
      #   url: uploads/resume.pdf
    design:
      background:
        gradient_mesh:
          enable: true
      name:
        size: md
      avatar:
        size: medium
        shape: circle

  - block: collection
    id: projects
    content:
      title: 项目
      text: ''
      count: 0
      filters:
        folders:
          - projects
        exclude_featured: false
      sort_by: 'Date'
      sort_ascending: false
    design:
      view: showcase
      columns: 1

  - block: resume-experience
    id: experience
    content:
      username: me
    design:
      date_format: 'Jan 2006'
      is_education_first: false
---
