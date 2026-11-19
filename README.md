from pygame import *
from random import randint
from time import time as timer 
font.init()
font1 = font.Font(None,80)
win = font1.render('YOU WIN!', True,(255,200,100))
losed = font1.render('YOU LOSE!', True,(180,0,0))
game = True
FPS = 100
clock = time.Clock()
class GameSprite(sprite.Sprite):
    def __init__(self,player_image,player_x,player_y,size_x ,size_y,player_speed,):
        super().__init__()
        self.image = transform.scale(image.load(player_image),(size_x ,size_y))
        self.speed = player_speed

        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
    def reset(self):
        window.blit(self.image, (self.rect.x,self.rect.y))

class Player(GameSprite):
    def update(self):
        keys = key.get_pressed()
        if keys[K_a] and self.rect.x > 5:
            self.rect.x -= self.speed
        if keys[K_d] and self.rect.x < win700 - 80:
            self.rect.x += self.speed   
    def fire(self):
        bullet = Bullet(img_bullet,self.rect.centerx, self.rect.top, 15,20,-15)
        bullets.add(bullet)
    

class Enemy(GameSprite):
    def update(self):
        self.rect.y += self.speed
        global lost
        if self.rect.y > win500:
            self.rect.x = randint(80,win700 - 80)
            self.rect.y = 0
            lost = lost + 1
class Bullet(GameSprite):
    def update(self):
        self.rect.y += self.speed
        if self.rect.y < 0:
            self.kill()        
score = 0
lost = 0
Finish = False
font.init()
font2 = font.Font(None,36)
text = font2.render('счет: '+ str(score), 1 , (255,200,100))
text_lose = font2.render('пропушено: '+ str(lost), 1 , (255,200,100))

mixer.init()
mixer.music.load('space.ogg')
mixer.music.play()
f_sound = mixer.Sound('fire.ogg')
win700 = 700
win500 = 500
window = display.set_mode((win700,win500))
display.set_caption('шутер')
background = transform.scale(image.load('galaxy.jpg'),(win700,win500))
ship = Player('rocket.png', 5 ,win500 - 100,80,100,5)
img_bullet = 'bullet.png'
img_enemy = 'ufo.png'

monsters = sprite.Group()
bullets = sprite.Group()
max_lose = 10
goal = 10  
asteroids = sprite.Group()

for i  in range(1,2):
    asteroid = Enemy('asteroid.png',randint(30,win700 - 30), -40,80,50,randint(1,7))
    asteroids.add(asteroid)

for i in range(1,5):
    monster = Enemy('ufo.png',randint(80,win700 - 80),-40,80,50,randint(1,2))
    monsters.add(monster)
life = 3
num_fire = 0
rel_time = False
last_time = 0
while game:

    
    for e in event.get():
        if e.type == QUIT:
            game = False 
        elif e.type == KEYDOWN:
            if e.key == K_SPACE:
                if num_fire < 5 and rel_time == False:
                    num_fire = num_fire +1
                    f_sound.play()
                    ship.fire()
                if num_fire >= 5 and rel_time == False:
                    last_time = timer()
                    rel_time = True
            if e.key == K_r:
                if last_time >= 5:
                    num_fire = 0 
                    rel_time = False
    if not Finish:
        window.blit(background,(0,0))
        window.blit(text,(10,20))
        window.blit(text_lose,(10,50))

        collides = sprite.groupcollide(monsters,bullets,True,True)
        for c in collides:
            score = score + 1
            monster = Enemy(img_enemy,randint(80,win700 - 80), -40,80,50, randint(1,5))
            monsters.add(monster)
        if sprite.spritecollide(ship,monsters,False) or sprite.spritecollide(ship , asteroids, False):
            sprite.spritecollide(ship,monsters,True)
            sprite.spritecollide(ship,asteroids,True)
            life = life - 1
        if life == 0 or lost >= max_lose:
            Finish = True
            window.blit(losed,(200,200))
        if score >= goal:
            Finish = True
            window.blit(win,(200,200))
        if life == 3:
            life_color = (0,140,0)
        if life == 2:
            life_color = (140,140,0)
        if life == 1:
            life_color = (140,0,0)
        text_life = font2.render('Жизней: '+ str(life), 1 , life_color)
        window.blit(text_life,(550,10))
        if rel_time == True:
            now_time = timer()

            if now_time - last_time < 3:
                reload = font2.render('ЭЙ ПОГОДИ ...', 1 , (150,0,0))
                window.blit(reload, (260,460))
            else:
                
                rel_time = False
        text = font2.render('счет: '+ str(score), 1 , (255,200,100))
        text_lose = font2.render('пропушено: '+ str(lost), 1 , (255,200,100))
        ship.update()
        ship.reset()
        asteroids.update()
        asteroids.draw(window ) 
        monsters.draw(window) 
        bullets.draw(window) 
        monsters.update() 
        bullets.update() 
        display.update()
    else:
        Finish = False
        score = 0
        lost = 0 
        num_fire = 0
        life = 3
        for b in bullets:
            b.kill()
        for m in monsters:
            m.kill()
        for a in asteroids:
            a.kill()
        clock.tick(3000)
        for i in range(1,6):
            monster = Enemy(img_enemy,randint(80,win700 - 80), -40,80,50, randint(1,5))
            monsters.add(monster)
        for i in range(1,6):
            asteroid = Enemy('asteroid.png',randint(80,win700 - 30), -40,80,50, randint(1,7))
            asteroids.add(asteroid)

        

    clock.tick(FPS)
