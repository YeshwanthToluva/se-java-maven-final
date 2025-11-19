git clone https://github.com/YeshwanthToluva/se-java-maven-final
cd se-java-maven-final
docker build -t spring-app-final:latest .  
docker run -d --name spring-app-run -p 7777:8080 spring-app-final:latest

