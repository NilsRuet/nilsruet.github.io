---
title: Research
summary:
type: landing

reading_time: false  # Show estimated reading time?
share: false  # Show social sharing links?
profile: false  # Show author profile?
comments: false  # Show comments?

sections:
  # A section to display my research
  - block: collection
    id: research-conference
    content:
      title: Conference/workshops
      subtitle: 
      text: 
      # Display content from the `content/post/` folder
      filters:
        folders:
          - "conference"
    design:
      # Choose how many columns the section has. Valid values: '1' or '2'.
      columns: '1'
      # Choose your content listing view - here we use the `showcase` view
      view: compact
      # For the Showcase view, do you want to flip alternate rows?
      flip_alt_rows: true

  - block: collection
    id: research-preprints
    content:
        title: Preprints
        subtitle: 
        text: 
        # Display content from the `content/post/` folder
        filters:
           folders:
            - "preprint"
    design:
        # Choose how many columns the section has. Valid values: '1' or '2'.
        columns: '1'
        # Choose your content listing view - here we use the `showcase` view
        view: compact
        # For the Showcase view, do you want to flip alternate rows?
        flip_alt_rows: true
---

