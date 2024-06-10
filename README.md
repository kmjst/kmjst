import pygame
from random import randint
import sys

pygame.init()
px,py=420,575
pan=pygame.display.set_mode([px,py])
pygame.display.set_caption("테트리스")
fps=pygame.time.Clock()

font1 = pygame.font.SysFont("Malgun Gothic",20)
font2 = pygame.font.SysFont("Malgun Gothic",40)

ds=25

cs=[(255,0,0),(255,128,0),(255,255,0),(0,255,0),
     (0,0,255),(0,255,255),(255,0,255)]

#tetromino tm
tm=[[[[1,1],[1,1]],[[1,1],[1,1]],[[1,1],[1,1]],[[1,1],[1,1]]],
    [[[0,0,0],[2,2,2],[0,2,0]],[[0,2,0],[2,2,0],[0,2,0]],
     [[0,2,0],[2,2,2],[0,0,0]],[[0,2,0],[0,2,2],[0,2,0]]],
    [[[0,0,0],[3,3,0],[0,3,3]],[[0,3,0],[3,3,0],[3,0,0]],
     [[3,3,0],[0,3,3],[0,0,0]],[[0,0,3],[0,3,3],[0,3,0]]],
    [[[0,0,0],[4,4,4],[0,0,4]],[[0,4,0],[0,4,0],[4,4,0]],
     [[4,0,0],[4,4,4],[0,0,0]],[[0,4,4],[0,4,0],[0,4,0]]],
    [[[0,0,0],[5,5,5],[5,0,0]],[[5,5,0],[0,5,0],[0,5,0]],
     [[0,0,5],[5,5,5],[0,0,0]],[[0,5,0],[0,5,0],[0,5,5]]],
    [[[0,0,0],[0,6,6],[6,6,0]],[[6,0,0],[6,6,0],[0,6,0]],
     [[0,6,6],[6,6,0],[0,0,0]],[[0,6,0],[0,6,6],[0,0,6]]],
    [[[0,0,0,0],[7,7,7,7],[0,0,0,0],[0,0,0,0]],
     [[0,0,7,0],[0,0,7,0],[0,0,7,0],[0,0,7,0]],
     [[0,0,0,0],[0,0,0,0],[7,7,7,7],[0,0,0,0]],
     [[0,7,0,0],[0,7,0,0],[0,7,0,0],[0,7,0,0]]]
    ]

m=[[0 for j in range(10)] for i in range(22)]
m[21]=[8 for j in range(10)]

def Ttransf(rt,rd):
    global tm,ln

    T=[]
    for i in range(ln):
        for j in range(ln):
            if tm[rt][rd][j][i] != 0 : T.append([i,j,rt+1])
    return T

def init_tet():
    global rt,rd,ln,rti,rdi,lni,tm,T,ydown,xp,yp,ypos,score
    
    rt,rd,ln=rti,rdi,lni
    xp=(5-(ln+1)//2)
    T=Ttransf(rt,rd)
    yp,ypos=0,25
    ydown=False

    rti,rdi=randint(0,6),randint(0,3)
    lni=len(tm[rti][rdi])
    score=score+rt+1

def draw():
    global ds,tm,rti,rdi,lni,m,cs,xp,yp,T

    pygame.draw.rect(pan,(0,0,0),(24,24,252,527),2)
    
    for i in range(lni):
        for j in range(lni):
            if tm[rti][rdi][i][j] != 0 :
                pygame.draw.rect(pan,cs[rti],(ds*(12+j)+2,ds*(2+i)+2,23,23))
                pygame.draw.rect(pan,(99,99,99),(ds*(12+j)+1,ds*(2+i)+1,24,24),1)


    for i in range(21):
        for j in range(10):
            if m[i][j] != 0:
                c=m[i][j]-1
                pygame.draw.rect(pan,cs[c],(ds*(j+1)+2,ds*(i+1)+2,23,23))
                pygame.draw.rect(pan,(99,99,99),(ds*(j+1)+1,ds*(i+1)+1,24,24),1)

    for i in range(4):
        x,y,c=xp+T[i][0]+1,yp+T[i][1]+1,T[i][2]-1
        pygame.draw.rect(pan,cs[c],(ds*x+2,ds*y+2,23,23))
        pygame.draw.rect(pan,(99,99,99),(ds*x+1,ds*y+1,24,24),1)


def update():
    global xp,yp,xm,ypos,vtet,rd,rot,m,T,ydown,ln

    #touch down check
    for i in range(4):
        x,y=xp+T[i][0],yp+T[i][1]
        if m[y+1][x] != 0:
            ydown=True
            renew_m()
            return
        
    #side touch
    for i in range(4):
        x,y=xp+T[i][0],yp+T[i][1]

        if xm<0 and x==0 : xm=0
        if xm<0 and x>0 :
            if m[y][x-1] != 0 : xm=0
        if xm>0 and x==9 : xm=0
        if xm>0 and x<9 :
            if m[y][x+1] != 0 : xm=0

    #rot touch
    if xp<=0 or xp>=10-ln : rot=0 
    if 0 <= xp <= 10-ln and yp < 21-ln:
        for i in range(ln):
            for j in range(ln):
                if m[yp+i][xp+j] != 0 : rot=0

    if rot == 1 :
        rd=(rd+rot)%4
        T=Ttransf(rt,rd)
        rot=0

    xp+=xm
    ypos+=vtet
    yp=ypos//25-1
    xm=0

def renew_m():
    global xp,yp,T,m,score

    for i in range(4):
        x,y,c=xp+T[i][0],yp+T[i][1],T[i][2]
        m[y][x]=c

    #del_line check & delete
    for i in range(21):
        if m[i].count(0) == 0 :
            score += sum(m[i])
            for k in range(i):
                im=i-k
                m[im][:]=m[im-1][:]

def godown():
    global xp,yp,m,T,ydn,ydown

    while True:
        for i in range(4):
            x,y=xp+T[i][0],yp+T[i][1]
            if m[y+1][x] != 0 :
                ydown=True
                renew_m()
                ydn=0
                return
        yp+=1

def game_over():

    Gover=font2.render("끝났다!",True,(255,55,55))
    again=font1.render("스페이스 바",True,(55,55,255))
    pan.blit(Gover,(280,400))
    pan.blit(again,(290,470))

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    
                    return
                  
        pygame.display.update()

xp,yp,xm,rot,ydn=0,0,0,0,0
rti,rdi=randint(0,6),randint(0,3)
lni=len(tm[rti][rdi])
ypos,vtet=25,5
ydown=True
T=[]
score=0

while True:

    pan.fill((220,220,220))
    scoreT=font1.render("점수 : "+str(score),True,(55,55,55))
    pan.blit(scoreT,(300,150))

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE: ydn=1
            if event.key == pygame.K_RIGHT: xm=1
            if event.key == pygame.K_LEFT: xm=-1
            if event.key == pygame.K_DOWN: rot=1

    vtet=min(9,5+score//500)
    if ydown: init_tet()
    update()
    if ydn==1 : godown()
    draw()
    
    #game_over
    if yp==0 and m[ln-1][5] != 0:
        game_over()
        
        m=[[0 for j in range(10)] for i in range(22)]
        m[21]=[8 for j in range(10)]
        score=0
        ydown=True

    pygame.display.update()
    fps.tick(20)
    
