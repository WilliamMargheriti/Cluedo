Code Cluedo

# -*- coding: utf-8 -*-
import sys
import random
from PyQt4 import QtGui
 
 # Taille des cases
resolution=25 

class Joueur():
    def __init__(self,string,ligdep,coldep,nlig,ncol,col):
        # Nom
        self.nom=string
        # Coordonnées de départ du joueur
        self.LD=ligdep
        self.CD=coldep
        # Coordonnées de la case actuelle du joueur
        self.LA=nlig
        self.CA=ncol
        # Couleur
        self.color=col
        
Rose=Joueur("Rose",0,15,0,15,"ff0000")
Moutarde=Joueur("Moutarde",5,21,5,21,"ffff00")
Leblanc=Joueur("Leblanc",21,15,21,15,"808080")
Olive=Joueur("Olive",21,6,21,6,"008000")
Pervenche=Joueur("Pervenche",17,0,17,0,"0000ff")
Violet=Joueur("Violet",3,0,3,0,"ee82ee")

Listejoueurs=[Rose,Moutarde,Leblanc,Olive,Pervenche,Violet]

class Piece():
    def __init__(self,string,ligdep,coldep,nlig,ncol,col):
        # Nom
        self.nom=string
        # Coordonnées de la case en haut à gauche de la pièce
        self.LD=ligdep
        self.CD=coldep
        # Dimensions de la pièce
        self.NL=nlig
        self.NC=ncol
        # Couleur
        self.color=col
        
Study=Piece("Study",0,0,3,6,"fff1f9")
Hall=Piece("Hall",0,8,6,6,"00f1f9")
Lounge=Piece("Lounge",0,16,4,6,"ffc3a0")
Library=Piece("Library",5,0,5,6,"daa520")
Billard=Piece("Billard Room",11,0,5,5,"6dc066")
Dining=Piece("Dining Room",7,15,7,7,"317332")    
Conservatory=Piece("Conservatory",18,0,4,5,"660066")       
Ballroom=Piece("Ballroom",16,7,6,8,"4169e1")
Kitchen=Piece("Kitchen",17,17,5,5,"d6e4c0")
Vide=Piece("",7,8,7,5,"000000")

Listepieces=[Study,Hall,Lounge,Library,Billard,Dining,Conservatory,Ballroom,Kitchen,Vide]
 
class Interface(QtGui.QWidget):
    def __init__(self):
        QtGui.QWidget.__init__(self)
        # Entier entre 0 et 5 qui désigne le joueur actif
        self.actif=0
        # Étape 0 : Début du tour, Étape 1 : Dé lancé, Étape 2 : Déplacement fait
        self.etape=0
        self.initUI()
 
    def initUI(self):
        # Création de la structure de grille de boutons
        self.grid = QtGui.QGridLayout()
        
        # On ajoute toutes les pièces à la grille
        for p in Listepieces:
            self.ajoutpiece(p)
        
        # Tableau 22*22 de 0
        self.Occup=[[0] * 22 for _ in range(22)]
        # On met un "1" dans les cases occupées par des pièces
        for p in Listepieces:
            for i in range(p.NL):
                for j in range(p.NC):
                    self.Occup[p.LD+i][p.CD+j]=1
          
        # Tableau de 22*22 boutons vides          
        self.button=[QtGui.QPushButton("")]*(22*22)
        
        for i in range(22):
            for j in range(22):
                # On crée un bouton simple blanc, occupant une case, pour toutes les cases non occupées
                if self.Occup[i][j]==0:
                    # Création d'un bouton
                    self.button[i+22*j]=QtGui.QPushButton("")
                    # Caractéristiques du bouton (couleur)
                    self.button[i+22*j].setStyleSheet("QPushButton{font-size:20px;background-color:#ffffff;border:1px} QPushButton:pressed{background-color:red;}")                   
                    # Caractéristiques du bouton (taille)                    
                    self.button[i+22*j].setFixedSize(resolution,resolution)
                    # Ajout du bouton sur la grille de boutons
                    self.grid.addWidget(self.button[i+22*j],i,j)
                    
        for i in range(22):
            for j in range(22):
                if self.Occup[i][j]==0:
                    # On définit l'action "Déplacement" liée au clic du bouton
                    #self.button[i+22*j].clicked.connect(lambda: self.Deplacement(i+22*j)) 
                    self.button[i+22*j].clicked.connect(self.Deplacement) 
        
        # Affichage des joueurs de la partie (en colorant les boutons sur les quels ils sont)
        for j in Listejoueurs:
            self.button[j.LA+22*j.CA].setStyleSheet("QPushButton{font-size:20px;background-color:#"+j.color+";border:1px} QPushButton:pressed{background-color:red;}")

         
        # Ajout d'un bouton Lancer de dé qui déclenche la fonction LancerDe
        self.dice = QtGui.QPushButton('Lancer le dé')
        self.dice.clicked.connect(self.LancerDe)
        self.grid.addWidget(self.dice,10,22,2,1)         
         
        # Ajout d'un bouton Tour suivant qui déclenche la fonction TourSuivant
        self.toursuivant = QtGui.QPushButton('Tour suivant')
        self.toursuivant.clicked.connect(self.TourSuivant)
        self.grid.addWidget(self.toursuivant,11,22,2,1) 

        # Ajout d'une barre d'état avec le lancer de dé
        self.barre=QtGui.QStatusBar()
        self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Veuillez lancer le dé")
        self.grid.addWidget(self.barre,22,0,22,24) 
         
        # Ligne nécessaire pour bien afficher la grille de boutons
        self.setLayout(self.grid)
      
        # Paramètres de la fenêtre      
        self.setWindowTitle('Grille pour CLUEDO')
        self.setGeometry(22*resolution+50,22*resolution+50,0,0)
        
    def ajoutpiece(self,Piece):
        # Affiche les pièces, ne sert que dans l'initialisation de la grille
    
        # Création d'un bouton
        self.button = QtGui.QPushButton(Piece.nom,self)
        # Caractéristiques du bouton (couleur)
        self.button.setStyleSheet("font-size:20px;background-color:#"+Piece.color+";border:none")
        # Caractéristiques du bouton (taille)
        self.button.setFixedSize(resolution*Piece.NC,resolution*Piece.NL)
        # Ajout du bouton sur la grille de boutons
        self.grid.addWidget(self.button,Piece.LD,Piece.CD,Piece.NL,Piece.NC)
     
    def Deplacement(self):   
        if self.etape==1:
            # On récupère l'adresse du bouton cliqué
            sender = self.sender()
            # Colonne
            i=(sender.x()-14)/27
            # Ligne
            j=(sender.y()-16)/27

            # Condition de déplacement, le nombre de cases doit être égal au lancer de dé 
            
            if (abs(i-Listejoueurs[self.actif].CA)+abs(j-Listejoueurs[self.actif].LA))==self.de:
                # On affiche le bouton du joueur en blanc
                self.button[Listejoueurs[self.actif].LA+22*Listejoueurs[self.actif].CA].setStyleSheet("QPushButton{font-size:20px;background-color:#ffffff;border:1px} QPushButton:pressed{background-color:red;}")
                # Nouvelles coordonnées du joueur actif
                Listejoueurs[self.actif].CA=i
                Listejoueurs[self.actif].LA=j
                # On colore le nouveau bouton du joueur
                self.button[j+22*i].setStyleSheet("QPushButton{font-size:20px;background-color:#"+Listejoueurs[self.actif].color+";border:1px} QPushButton:pressed{background-color:red;}")
                self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Résultat du lancer de dé : "+str(self.de))
                self.etape=2
            else:
                # On affiche sur la barre d'état "Cliquez sur une autre case"
                self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Résultat du lancer de dé : "+str(self.de)+" | Veuillez cliquer sur une autre case")
        elif self.etape==0:
            self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Résultat du lancer de dé : "+str(self.de)+" | Veuillez lancer le dé")
        else:
            self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Résultat du lancer de dé : "+str(self.de)+" | Veuillez finir votre tour")
    def LancerDe(self):
        if self.etape==0:
            # On tire un entier au hasard
            self.de=random.randint(1,6)
            # On l'affiche dans la barre d'état
            self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Résultat du lancer de dé : "+str(self.de))
            self.etape=1
        elif self.etape==1:
            self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Résultat du lancer de dé : "+str(self.de)+" | Veuillez cliquer sur une case pour vous déplacer")
        else:
            self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Résultat du lancer de dé : "+str(self.de)+" | Veuillez finir votre tour")
            
    def TourSuivant(self):
        if self.etape==2:
            # On passe au joueur suivant (0->1, 1->2, etc...)
            self.actif=(self.actif+1)%6
            # On affiche le joueur actif sur la barre d'état
            self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Résultat du lancer de dé : "+str(self.de))
            self.etape=0
        elif self.etape==1:
            self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Résultat du lancer de dé : "+str(self.de)+" | Veuillez cliquer sur une case pour vous déplacer")
        else:
            self.barre.showMessage("Joueur actif : "+Listejoueurs[self.actif].nom+" | "+"Résultat du lancer de dé : "+str(self.de)+" | Veuillez lancer le dé")

def main():
    app = QtGui.QApplication(sys.argv)
    main= Interface()
    main.show()
 
    sys.exit(app.exec_())
 
if __name__ == "__main__":
    main()
