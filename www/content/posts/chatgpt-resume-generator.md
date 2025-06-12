---
title: "Generate Custom Resumes with ChatGPT"
summary: "Use ChatGPT to create custom tailored resumes based on a given job description"
tags: ["ChatGPT", "AI", "automation", "job search"]
date: 2025-05-25
draft: false
---

# Introduction

Looking for work in the current tech job market has never been more frustrating. If you have a broad range of experience, you're likely qualified for several roles. That means maintaining multiple resumes on top of needing to tailor them per job application. With a little bit of prep, ChatGPT can be configured to generate quality resumes based on a given job description to save time instead of manually juggling and editing resumes. 

# Getting Started

I use the following tools:
- **[ChatGPT app](https://openai.com/chatgpt/download/) for macOS** (the Windows and browser versions of ChatGPT do not support integrating with other apps*)
- **[VS Code](https://code.visualstudio.com/)** (or any text editor of your choice)
- **Microsoft Word** (used to convert the final HTML into a PDF)

The goal is to have ChatGPT generate a custom resume in HTML within VS Code that can be saved as a PDF using Word. HTML is used because it offers more control over styling and layout when converted to PDF.

**If you're unable to use the ChatGPT macOS app, you can still copy/paste HTML between ChatGPT and VS Code. You could use [Cursor](https://www.cursor.com/) as an alternative to both ChatGPT plus VS Code on Windows or macOS.*

# Initial Setup

Some initial prep work is needed to get ChatGPT to generate good results. ChatGPT works best when given context and constraints so I created the following files to work with:
- **CV.json**, my curriculum vitae
- **prompt.json**, a set of defined tasks plus lists of explicit does and dont's for ChatGPT

I use JSON because it allows ChatGPT to precisely parse data.

## CV.json

Put as much information as you can for ChatGPT to draw from. Modify the JSON however you want by adding/removing fields. Use ChatGPT to help if you're not familiar with JSON. The goal is to give ChatGPT as much context to draw from to match job descriptions.

JSON example:
```json
{
  "name": "John Doe",
  "title": "IT Professional",
  "summary": "Experienced IT professional with a focus on systems administration and cloud technologies.",
  "contact": {
    "email": "john.doe@example.com",
    "phone": "(000) 000-0000",
    "linkedin": "https://www.linkedin.com/in/johndoe",
    "website": "https://www.johndoe.com",
    "location": "City, State"
  },
  "skills": [
    "Skill 1",
    "Skill 2"
  ],
  "tools": [
    "Tool 1",
    "Tool 2"
  ],
  "certifications": [
    {
      "name": "Certification Name",
      "issuer": "Issuing Organization",
      "year": 2024
    }
  ],
  "certifications_in_progress": [
    {
      "name": "Certification Name",
      "issuer": "Issuing Organization",
      "estimated_completion": "Spring 2025"
    }
  ],
  "experience": [
    {
      "title": "Job Title",
      "company": "Company Name",
      "location": "City, State",
      "work_mode": "remote",
      "start_date": "Month/Year",
      "end_date": "Month/Year",
      "company_description": "Brief company description.",
      "bullets": [
        "Responsibility or accomplishment one.",
        "Responsibility or accomplishment two."
      ],
      "tools_used": [
        "Tool A",
        "Tool B"
      ],
      "achievements": [
        "Achievement description."
      ]
    }
  ],
  "personal_projects": [
    {
      "name": "Project Name",
      "description": "Brief description of the personal project.",
      "technologies": [
        "Technology A",
        "Technology B"
      ],
      "link": "https://www.projectlink.com"
    }
  ],
  "education": [
    {
      "school": "School Name",
      "degree": "Degree or Diploma",
      "location": "City, State"
    }
  ],
  "more_info": {
    "preferred_tools": [
      "Tool A",
      "Tool B"
    ],
    "preferred_roles": [
      "Role A",
      "Role B"
    ]
  }
}
```

## prompt.json

Anyone experienced with AI knows it can hallucinate so this is to prevent that. This is also where you can give ChatGPT precise instructions on how to format the resume. I would avoid having ChatGPT try to create anything overly stylized. My understanding is simple resumes are better for ATS parsing so avoid colors, tables, etc. 

JSON example:
```json
{
    "task": [
      "You are a resume and cover letter generating assistant.",
      "You are given my curriculum vitae in JSON format.",
      "You are given a job description via URL or pasted into chat and will create a custom resume tailored to it.",
      "You will output in HTML with complex styling into resume.html using integration with VS Code keeping in mind that it will be converted to PDF using Word.",
      "You are given lists of do's and don'ts to explicitly follow.",
      "You may also be asked to generate resumes based on conversations with me instead of a specific job description."
    ],
    "do": [
      "Use only content from the curriculum vitae JSON.",
      "Format as a 2-page single column resume using a max of 15 bullets for experience across all roles.",
      "Keep summary to 3 sentences and have it align with job description noting matching skills",
      "Always structure the resume in this order: Summary, Certifications, Professional Experience & accomplishments, Skills, Projects (if relevant), Education.",
      "List certifications and certifications in progress only if relevant to the job description.",
      "Include one project only if it demonstrates direct experience with tools or practices directly mentioned in the job description.",
      "Ensure the resume is optimized to pass ATS filters. No use of colors, tables, or other formatting that may not parse.",
      "If listing certifications in progress only list one most relevant to the job.",
      "Focus on preferred tools as defined in full job history JSON",
      "Include 1-2 accomplishments for each employer that are most relevant to the job description. If there are none given in full job history JSON, create one based on bullets without any embellishment. Include as bulletized subsection under experience.",
      "Follow up asking to generate cover letter HTML after generating resume HTML.",
      "Suggest edits of the 2 JSON files as necessary to generate better content."
],
    "do_not": [
      "Invent skills, tools, or experience not found in the job history JSON.",
      "Invent stats or metrics not found in the job history JSON.",
      "Do not add things from the job description that weren't in the job history JSON even if acknowledging it.",
      "Use filler language, fluff, em dashes, emoji, or anything to give away that AI is composing.",
      "Mention personal projects unless they're relevant to the job description",
      "Make things up if unable to visit a URL, ask for pasted description.",
      "Duplicate experience bullets by also making them accomplishments."
    ]
  }
```

# Working with ChatGPT

I start a new chat with the two JSON files attached and give the ChatGPT app access to a blank resume.html file in VS Code. Then I provide either a pasted job description or a URL to one, along with the prompt below.

> You have been given 2 JSON files. “prompt.json” defines your tasks as a resume generating assistant with lists of things to do and not do. “CV.json” is my curriculum vitae for you to use to complete the task. Do not deviate from what’s defined in “prompt.json”.

ChatGPT generates a custom resume written in HTML within VS Code. I save the HTML file and then view it using a web browser and edit as needed in VS Code or by prompting ChatGPT to edit. Once satisfied, I open it with Word and save it as a PDF.

[View Example Resume HTML](/assets/posts/chatgpt-resume-generator/example-resume.html)

[View Example Resume HTML Code](https://github.com/scottlowry/ScottLowryNet/blob/main/www/static/assets/posts/chatgpt-resume-generator/example-resume.html)

[Download Example Resume PDF](/assets/posts/chatgpt-resume-generator/example-resume.pdf)

## Tips & Tricks

You can have ChatGPT generate data for fields in CV.json such as title and summary to match job descriptions. 

Edit CV.json and replace fields you want generated with [GENERATE]:
```json
{
"name": "John Doe",
"title": "[GENERATE]",
"summary": "[GENERATE]"
}
```

Also edit prompt.json, adding a new line under "do" section to generate content for fields set as [GENERATE]:
```json
{
"do": [
"Generate content for key values in the curriculum vitae JSON set to [GENERATE] to match the job description.",
"Other things to do..."
]
}
```

# Conclusion

***Always thoroughly check ChatGPT's output!*** The guardrails set in prompt.json should prevent hallucinations but you should *always* double check. Make sure formatting is ok too, especially after using Word to save as PDF.