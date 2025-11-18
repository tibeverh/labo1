- Student name: Tibe Verhoestraete 
- GitHub repo: https://github.com/tibeverh/labo1.git


## 1.1 set up the lab environment

- Ga naar de juiste folder met het `cd` commando.

- Voer het commando `vagrant up` vervolgens uit.

- Daarna doe je  het commando `vagrant ssh`.

- check of er al een container met Portainer running is gebruik het commando `sudo docker ps`

- ga naar een webbrowser op je fysieke systeem en surf naar <http://192.168.56.20:9000/>
![alt text](<Schermopname (2242).png>)

## 1.1 git configuren

- geef commando `git config --global --list` om te kijken of `user.name` en `user.email` ingesteld zijn.
- kopiëer de code van `cicd-sample-app` naar een nieuwe directory buiten de huidige git directory.
- gebruik het commando `git init` en commit alle code
- creëer een nieuwe public repository en record de URL
- link je local repository met degene die je hebt gemaakt op github met het commando `git remote add origin git@github.com:USER/cicd-sample-app.git` (USER vervangen door Github naam)
- push de local committed code naar Github met het commando `git push -u origin main`

![alt text](<Schermopname (2243).png>)


## 1.2 Build and verify the sample application

- ga naar de juiste map met het `cd` commando.

- geef het commando `vagrant ssh` in om te verbinden met de VM
- ![alt text](<Schermopname (2245).png>)

- ga naar de directory `/vagrant/cicd-sample-app` met het commando ´cd /vagrant/cicd-sample-app´
![alt text](<Schermopname (2246).png>)

- voer het script sample-app.shh uit met het commando `./sample-app.sh`.

- verifiëer of de app runt door een browser op je fysieke toestel te openen en te surfen naar <http://192.168.56.20:5050/>
![alt text](image-5.png)

- stop en verwijder portainer container met de volgende commandos `docker stop portainer` & `docker rm portainer`
- 
- kijk als het gelukt is met het commando `docker ps`
  

## 1.3 Download and run the Jenkins Docker image

- Download de Jenkins image met het commando `docker pull jenkins/jenkins:lts`

- start de jenkins docker container door het volgende commando volledig uit te voeren in 1 keer
  
 ```console
    docker run -p 8080:8080 -u root \
      -v jenkins-data:/var/jenkins_home \
      -v $(which docker):/usr/bin/docker \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v "$HOME":/home \
      --name jenkins_server jenkins/jenkins:lts
 ```

- als dit gelukt is zal je uw wachtwoord zien
![alt text](image-6.png)

- Mijn wachtwoord is 57d7ab3e1c2d4f7d8671f87accf5ffb3.
- Als je je wachtwoord vergeten bent dan kan je het terug vinden met het volgende commando `docker exec -it jenkins_server /bin/cat /var/jenkins_home/secrets/initialAdminPassword`

## 1.4 Configure Jenkins

- open een browser en surf naar <http://192.168.56.20:8080/>.
  
- Druk op suggested plugins.

- Kies skip and continue as admin.
 
- Druk op Save and Finish.

- Druk op Start using Jenkins.

## 1.5 Use Jenkins to build your application

- Op het dashboard druk je op create job en geef je een naam in en kies je voor Freestyle project
![alt text](<Schermopname (2250).png>)

- Bij broncodebeheer vink je Git aan en geef je je repository url in.
![alt text](<Schermopname (2251).png>)

- Bij branches to be build verander je "*/master" naar "*/main"
![alt text](<Schermopname (2252).png>)

- Bij Build Steps"druk je op Add Build Step en kies je voor Execute Shell.
- 
- geef code in dit is anders dan in de opdracht maar anders lukte het niet.
![alt text](<Schermopname (2253).png>)

- Bij de job druk je op build now
![alt text](<Schermopname (2254).png>)


## 1.6
IP adres vinden voor sampleapp en jenkins_server met het commando:
- `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' samplerunning` voor sampleapp container
- `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' jenkins_server` voor jenkins container

c:\users\tibev\OneDrive\Afbeeldingen\Schermopnamen\Schermopname (2255).png

- JENKINS_IP 172.17.0.2
- APP_IP 172.17.0.3

- kijk of de app beschikbaar is met het commando `curl http://172.17.0.2:5050/`

- maak een nieuwe job aan.

c:\users\tibev\OneDrive\Afbeeldingen\Schermopnamen\Schermopname (2256).png

- ga naar de sectie "Build Triggers" en selecteer de checkbox Build after other projects are built
- vul in het veld Projects to watch de naam in van je build job in
- ga naar de sectie Build Steps en kies voor een build step van het type Execute Shell
- geef de code van in de foto in
![alt text](<Schermopname (2257).png>)
![alt text](<Schermopname (2258).png>)
![alt text](<Schermopname (2259).png>)

- save en run de job om te verifiëeren of het werkt

![alt text](<Schermopname (2260).png>)


## 1.7 Create a build pipeline 

- klik bij het dashbord op Create New Item

- ga naar de sectie "Pipeline" en zet volgende code in het tekstveld:

     ```text
    node {
        stage('Preparation') {
            catchError(buildResult: 'SUCCESS') {
                sh 'docker stop samplerunning'
                sh 'docker rm samplerunning'
            }
        }
        stage('Build') {
            build 'BuildSampleApp'
        }
        stage('Results') {
            build 'TestSampleApp'
        }
    }

    ```

![alt text](<Schermopname (2261).png>)

- Druk op build now en kijk als het werkt
  
## 1.8 Use a Jenkinsfile

- Voeg in github een jenkinsfile met de volgende code toe:
  ```text
    node {
        stage('Preparation') {
            catchError(buildResult: 'SUCCESS') {
                sh 'docker stop samplerunning'
                sh 'docker rm samplerunning'
            }
        }
        stage('Build') {
            build 'BuildSampleApp'
        }
        stage('Results') {
            build 'TestSampleApp'
        }
    }

    ```
![alt text](<Schermopname (2262).png>)

- verwijder de pipeline van het Jenkins dashbord
- create een nieuwe pipeline voor de Git repository:
    - klik op "Create new Item"
    - geef een gepaste naam in (bv. SampleAppPipeline)
    - kies voor "Pipeline" als job type
    - ga naar de sectie "Pipeline" en kies voor de optie "Pipeline script from CSM" bij "Definition"
    - kies voor "Git" bij "CSM" en geef de URL van je GitHub repository in
    - kies voor de main branch bij "Branch specifier" e, druk op de knop "Save"

![alt text](<Schermopname (2263).png>)
![alt text](<Schermopname (2264).png>)
![alt text](<Schermopname (2265).png>)

- Druk op build now en kijk als het werkt
![alt text](<Schermopname (2265)-1.png>)

## 1.9 Make a change in the application

- ga in de Git repository van de applicatie en open het bestand `static/style.css`
- verander de achtergrond kleur in bv geel
- save het bestand, commit je veranderingen en push het naar github
- launch de build pipeline op het Jenkins dashbord
- herlaad de applicatie in de browser, deze moet een andere achtergrond kleur krijgen

![alt text](<Schermopname (2266).png>)
![alt text](<Schermopname (2267).png>)



## cheatsheet
| Task                                           | Command                                                                                          |
| :---                                           | :---                                                                                             |
| git configuratie bekijken                      | `git config --global --list`                                                                     |
| bestanden toevoegen om te committen            | `git add FILE...`                                                                                |
| changes committen naar local repository        | `git commit -m 'MESSAGE'`                                                                        |
| local changes pushen naar remote repository    | `git push`                                                                                       |
| changes van remote repo naar local repo pullen | `git pull`                                                                                       |
| docker container stoppen                       | `docker stop ...`                                                                                |
| docker container verwijderen                   | `docker rm ...`                                                                                  |
| docker containers bekijken                     | `docker ps`                                                                                      |
| ip adres van een container achterhalen         | `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' samplerunning ...` |
| docker installeren                             | `docker pull jenkins/jenkins:lts`                                                                |
| virtuele machine booten met vagrant            | `vagrant up  ...`                                                                                |
| verbinden met virtuele machine met vagrant     | `vagrant up  ...`                                                                                |