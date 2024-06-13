# Assembly Code to documentation

## Introduction

- How to convert assembly code to documentation
- parse existing x86 assembly code
- convert to python
- Create documentation
- Ask the model to explain
- Using sample from - https://cs.lmu.edu/~ray/notes/x86assembly/

## Steps

- I am using my sample app to test with GPT 4o model
- Here is my sample prompt with sample code

```
Create documentation for the below code:
        global  _main
        extern  _glClear@4
        extern  _glBegin@4
        extern  _glEnd@0
        extern  _glColor3f@12
        extern  _glVertex3f@12
        extern  _glFlush@0
        extern  _glutInit@8
        extern  _glutInitDisplayMode@4
        extern  _glutInitWindowPosition@8
        extern  _glutInitWindowSize@8
        extern  _glutCreateWindow@4
        extern  _glutDisplayFunc@4
        extern  _glutMainLoop@0

        section .text
title:  db      'A Simple Triangle', 0
zero:   dd      0.0
one:    dd      1.0
half:   dd      0.5
neghalf:dd      -0.5

display:
        push    dword 16384
        call    _glClear@4              ; glClear(GL_COLOR_BUFFER_BIT)
        push    dword 9
        call    _glBegin@4              ; glBegin(GL_POLYGON)
        push    dword 0
        push    dword 0
        push    dword [one]
        call    _glColor3f@12           ; glColor3f(1, 0, 0)
        push    dword 0
        push    dword [neghalf]
        push    dword [neghalf]
        call    _glVertex3f@12          ; glVertex(-.5, -.5, 0)
        push    dword 0
        push    dword [one]
        push    dword 0
        call    _glColor3f@12           ; glColor3f(0, 1, 0)
        push    dword 0
        push    dword [neghalf]
        push    dword [half]
        call    _glVertex3f@12          ; glVertex(.5, -.5, 0)
        push    dword [one]
        push    dword 0
        push    dword 0
        call    _glColor3f@12           ; glColor3f(0, 0, 1)
        push    dword 0
        push    dword [half]
        push    dword 0
        call    _glVertex3f@12          ; glVertex(0, .5, 0)
        call    _glEnd@0                ; glEnd()
        call    _glFlush@0              ; glFlush()
        ret

_main:
        push    dword [esp+8]           ; push argv
        lea     eax, [esp+8]            ; get addr of argc (offset changed :-)
        push    eax
        call    _glutInit@8             ; glutInit(&argc, argv)
        push    dword 0
        call    _glutInitDisplayMode@4
        push    dword 80
        push    dword 80
        call    _glutInitWindowPosition@8
        push    dword 300
        push    dword 400
        call    _glutInitWindowSize@8
        push    title
        call    _glutCreateWindow@4
        push    display
        call    _glutDisplayFunc@4
        call    _glutMainLoop@0
        ret
```
  
- here is the output
  
![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/assembly2.jpg 'RagChat')
![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/assembly3.jpg 'RagChat')
![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/assembly4.jpg 'RagChat')
![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/assembly5.jpg 'RagChat')


```
Simple Triangle OpenGL Program Documentation
Overview
This assembly program initializes an OpenGL context using GLUT and renders a simple colored triangle. The triangle is drawn using three vertices, each with a different color, creating a gradient effect.

Sections
Global and External Declarations
global  _main
extern  _glClear@4
extern  _glBegin@4
extern  _glEnd@0
extern  _glColor3f@12
extern  _glVertex3f@12
extern  _glFlush@0
extern  _glutInit@8
extern  _glutInitDisplayMode@4
extern  _glutInitWindowPosition@8
extern  _glutInitWindowSize@8
extern  _glutCreateWindow@4
extern  _glutDisplayFunc@4
extern  _glutMainLoop@0

global _main: Entry point of the program.
extern: External functions from OpenGL and GLUT libraries.
Data Section
section .text
title:  db      'A Simple Triangle', 0
zero:   dd      0.0
one:    dd      1.0
half:   dd      0.5
neghalf:dd      -0.5

title: Window title string.
zero, one, half, neghalf: Constants used for vertex positions and colors.
Display Function
display:
        push    dword 16384
        call    _glClear@4              ; glClear(GL_COLOR_BUFFER_BIT)
        push    dword 9
        call    _glBegin@4              ; glBegin(GL_POLYGON)
        push    dword 0
        push    dword 0
        push    dword [one]
        call    _glColor3f@12           ; glColor3f(1, 0, 0)
        push    dword 0
        push    dword [neghalf]
        push    dword [neghalf]
        call    _glVertex3f@12          ; glVertex(-.5, -.5, 0)
        push    dword 0
        push    dword [one]
        push    dword 0
        call    _glColor3f@12           ; glColor3f(0, 1, 0)
        push    dword 0
        push    dword [neghalf]
        push    dword [half]
        call    _glVertex3f@12          ; glVertex(.5, -.5, 0)
        push    dword [one]
        push    dword 0
        push    dword 0
        call    _glColor3f@12           ; glColor3f(0, 0, 1)
        push    dword 0
        push    dword [half]
        push    dword 0
        call    _glVertex3f@12          ; glVertex(0, .5, 0)
        call    _glEnd@0                ; glEnd()
        call    _glFlush@0              ; glFlush()
        ret

display: Function to render the triangle.
glClear(GL_COLOR_BUFFER_BIT): Clears the screen.
glBegin(GL_POLYGON): Begins drawing a polygon.
glColor3f: Sets the color for the next vertex.
glVertex3f: Specifies a vertex.
glEnd: Ends the polygon drawing.
glFlush: Flushes the OpenGL commands to ensure they are executed.
Main Function
_main:
        push    dword [esp+8]           ; push argv
        lea     eax, [esp+8]            ; get addr of argc (offset changed :-)
        push    eax
        call    _glutInit@8             ; glutInit(&argc, argv)
        push    dword 0
        call    _glutInitDisplayMode@4
        push    dword 80
        push    dword 80
        call    _glutInitWindowPosition@8
        push    dword 300
        push    dword 400
        call    _glutInitWindowSize@8
        push    title
        call    _glutCreateWindow@4
        push    display
        call    _glutDisplayFunc@4
        call    _glutMainLoop@0
        ret

_main: Entry point of the program.
glutInit: Initializes GLUT.
glutInitDisplayMode: Sets the display mode.
glutInitWindowPosition: Sets the initial window position.
glutInitWindowSize: Sets the initial window size.
glutCreateWindow: Creates a window with the specified title.
glutDisplayFunc: Registers the display callback function.
glutMainLoop: Enters the GLUT event processing loop.
Summary
This program sets up an OpenGL context using GLUT and renders a simple triangle with each vertex having a different color. The triangle is displayed in a window titled "A Simple Triangle". The main function initializes the necessary GLUT settings and registers the display function, which is responsible for rendering the triangle.
```