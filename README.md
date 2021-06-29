;例子１
;01H 从键盘读一字符并在屏幕显示
;---------------------------
code segment public
assume cs:code,ds:code
    org 100h
start:    jmp begin
msg1    db 'Press any key to test,Ctrl-Z to end.',0dh,0ah,'$'
msg2    db 'Normal Ascii Character',0dh,0ah,'$'
msg3    db 'Extended Ascii Character',0dh,0ah,'$'

begin:    mov ax,cs
    mov ds,ax
    mov dx,offset msg1    ;set up to display message
    mov ah,09h            ;display string function request
    int 21h               ;call DOS
next:
    mov ah,01h            ;read keyboard and echo function request
    int 21h               ;call DOS
    mov cx,offset msg2    ;set up to display message
    cmp al,0              ;check if extended ascii char
    jne disp              ;no, tack jump
    mov ah,01h            ;read keyboard and echo function request
    int 21h               ;call DOS
    mov cx,offset msg3    ;set up to display message
disp:
    cmp al,1ah            ;check for control-z
    je done               ;quit if control-z
    mov dl,al             ;return char in dl for next function
    mov ah,02h            ;display character funct request
    int 21h               ;call DOS
    mov dx,cx             ;get proper message
    mov ah,09h            ;display string function request
    int 21h               ;call DOS
    jmp next              ;get next character 
done:
    mov ah,00h            ;terminate progream funct request
    int 21h               ;call DOS
code ends                 ;end of code segment
end start                 ;start is the entry point
;-----------------------------
;例子２：
;07H 执行直接控制台输入但不回显
;此功能调用不检查字符是否是Ctrl-Break或Ctrl-PrtSc且不对这些字符做特殊处理。
;带Ctrl-Break检查、执行相同功能的，参见功能调用08。

;------------------------------
code segment public
assume cs:code, ds:code
    org 100h
start:    jmp begin
msg1    db 'Type anything,followed by enter; type 0 to quit.',13,10,'$'
msg2    db 'YOUR SECRET MESSAGE WAS: ','$'
secret db 40 dup(?)

begin:
    mov ax,cs
    mov ds,ax
    mov dx,offset msg1
    mov ah,09h
    int 21h

getChar:
    mov cx,39
    xor bx,bx
next:
    mov ah,07h
    int 21h
    cmp al,0dh
    je disp
    cmp al,30h
    je quit
    mov secret[bx],al
    inc bx
    loop next
disp:
    mov secret[bx],13
    inc bx
    mov secret[bx],10
    inc bx
    mov secret[bx],'$'
    
    ;mov dx,offset msg2
    ;mov ah,09h
    ;int 21h
    mov dx,offset secret
    mov ah,09h
    int 21h
    jmp getChar

quit:    
    mov ah,00h
    int 21h
    
code ends
end start
;-----------------------
;例子３
08H 从键盘读一字符但不回显，此功能调用检查字符是否是Ctrl-Break。
;----------------------
code segment public
assume cs:code,ds:code
    org 100h
start:  jmp begin
msg1    db 'Type a secret message, followed by enter.',0dh,0ah,'$'
msg2    db 'YOUR SECRET MESSAGE WAS: ','$'
secret   db 40 dup (?)
begin:  mov ax,cs
    mov ds,ax
    mov dx,39
    xor bx,bx
next:    mov ah,08h
    int 21h
    cmp al,0dh
    je disp
    mov secret[bx],al
    inc bx
    loop next
disp: mov secret[bx],'$'
    mov dx,offset msg2
    mov ah,09h
    int 21h
    mov dx, offset secret
    mov ah,09h
    int 21h
    mov ax,4c00h
    int 21h
code ends
    end start 