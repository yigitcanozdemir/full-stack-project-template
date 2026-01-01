# Full Stack Project Template

This repository serves as a template for building full-stack applications with a Python backend and a frontend.

## Adjusting .gitignore
Ensure you adjust the .gitignore file according to your project needs. For example, since this is a template, the /data/ folder is commented out and data will not be excluded from source control:

```gitignore
# exclude data from source control by default
# /data/
```
Typically, you want to exclude this folder if it contains either sensitive data that you do not want to add to version control or large files.

## Duplicating the .env File
To set up your environment variables, you need to duplicate the .env.example file and rename it to .env. You can do this manually or using the following terminal command:

```bash
cp .env.example .env # Linux, macOS, Git Bash, WSL
copy .env.example .env # Windows Command Prompt
```
This command creates a copy of .env.example and names it .env, allowing you to configure your environment variables specific to your setup.

## Project Organization
```
├── LICENSE            <- Open-source license if one is chosen
├── README.md          <- The top-level README for developers using this project.
├── docker-compose.yml <- Docker Compose configuration for the entire stack.
├── .env.example       <- Example environment variables file.
├── .gitignore         <- Specifies intentionally untracked files to ignore.
│
├── backend            <- Backend source code (Python).
│   ├── main.py        <- Entry point for the backend application.
│   ├── config.py      <- Configuration management and useful variables.
│   ├── requirements.txt <- The requirements file for reproducing the backend environment.
│   ├── api            <- API route definitions and logic.
│   ├── db             <- Database connection and interaction logic (e.g., Postgres).
│   ├── modeling       <- Scripts for data modeling or AI components.
│   ├── services       <- Service classes to connect with external platforms, tools, or APIs.
│   └── utility        <- Utility functions and helper scripts.
│
└── frontend           <- Frontend source code.
    └── Dockerfile     <- Dockerfile for the frontend.
```