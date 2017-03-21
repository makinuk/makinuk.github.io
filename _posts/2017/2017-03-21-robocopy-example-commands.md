---
layout: post
title: "Robocopy Example Commands"
published: true
date: 2017-03-21 10:00:00
categories: [Robocopy]
excerpt: | 
        Robocopy is a command that is used at the command line to make copies of files and folders. 
        
        It is also known as Robust File Copy. It comes with windows vista, but was also part of the windows resource kit. It was made to be used for mirroring directories.
---

## Robocopy Examples

Robocopy is a command that is used at the command line to make copies of files and folders. It is also known as Robust File Copy. It comes with windows vista, but was also part of the windows resource kit. It was made to be used for mirroring directories.

[Official Documentation](https://technet.microsoft.com/en-us/library/cc733145(v=ws.11).aspx#Syntax)

### Move all folders and files

```
SET MoveDirSource=\\path\to\source\
SET MoveDirDestination=D:\path\to\target

MKDIR "%MoveDirDestination%"
FOR    %%i IN ("%MoveDirSource%\*") DO           MOVE /Y "%%i" "%MoveDirDestination%\%%~nxi"
FOR /D %%i IN ("%MoveDirSource%\*") DO ROBOCOPY /MOVE /E "%%i" "%MoveDirDestination%\%%~nxi"
```
