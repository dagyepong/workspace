## Introduction to Open-Resume
OpenResume is a powerful open-source resume builder and resume parser.

The goal of OpenResume is to provide everyone with free access to a modern professional resume design and enable anyone to apply for jobs with confidence.

OpenResume’s second component is the resume parser. For those who have an existing resume, the resume parser can help test and confirm its ATS readability.

## Docker compose Install
Open resume can be installed with docker compose and below is sample compose.yml file
```yml
version: '3.9'
services:
    open-resume:
        image: 'itsnoted/open-resume:latest'
        ports:
            - '3038:3000'
```
To access the web ui, got to http://ip:3038.

??? note

    Open Resume can be used to share your resume on the go to recruiters.
    This makes it some much easy for professionals to save time and effort on creating a new resume for a career change
    Feel free to change the ports incase 3038 is already mapped to another container.
