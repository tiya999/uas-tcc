Tugas 1: Jalankan beberapa wadah Docker sederhana
Ada berbagai cara untuk menggunakan wadah. Ini termasuk:

Untuk menjalankan satu tugas: Ini bisa berupa skrip shell atau aplikasi khusus.
Interaktif: Ini menghubungkan Anda ke wadah yang mirip dengan cara Anda SSH ke server jauh.
Di latar belakang: Untuk layanan jangka panjang seperti situs web dan basis data.
Di bagian ini Anda akan mencoba masing-masing opsi tersebut dan melihat bagaimana Docker mengelola beban kerja.

Jalankan satu tugas dalam wadah Alpine Linux
Pada langkah ini kami akan memulai sebuah wadah baru dan memintanya untuk menjalankan perintah nama host. Kontainer akan mulai, jalankan perintah hostname, lalu keluar.
1. Jalankan perintah berikut di konsol Linux Anda.
    docker container run alpine hostname
    Output di bawah ini menunjukkan bahwa alpine: gambar terbaru tidak dapat ditemukan secara lokal. Ketika ini terjadi, Docker secara otomatis menariknya dari Docker Hub.

2. Docker menjaga wadah berjalan selama proses itu dimulai di dalam wadah masih berjalan. Dalam hal ini proses nama host keluar segera setelah output ditulis. Ini artinya wadah berhenti. Namun, Docker tidak menghapus sumber daya secara default, sehingga wadah masih ada dalam status Keluar.
    docker container ls --all

Jalankan wadah Ubuntu interaktif
Anda dapat menjalankan sebuah wadah berdasarkan versi Linux yang berbeda dari yang dijalankan pada host Docker Anda.

Pada contoh berikut, kita akan menjalankan wadah Ubuntu Linux di atas host Alpine Linux Docker (Play With Docker menggunakan Alpine Linux untuk node-node-nya).

1. Jalankan wadah Docker dan akses cangkangnya.
 docker container run \
 --detach \
 --name mydb \
 -e MYSQL_ROOT_PASSWORD=my-secret-pw \
 mysql:latest

 --detach will run the container in the background.
--name will name it mydb.
-e will use an environment variable to specify the root password (NOTE: This should never be done in production).
2. List the running containers.
docker container ls
3. You can check what’s happening in your containers by using a couple of built-in Docker commands: docker container logs and docker container top.
 docker container logs mydb
4. List the MySQL version using docker container exec.
docker container exec allows you to run a command inside a container. In this example, we’ll use docker container exec to run the command-line equivalent of mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version inside our MySQL container.
docker exec -it mydb \
 mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version
5. You can also use docker container exec to connect to a new shell process inside an already-running container. Executing the command below will give you an interactive shell (sh) inside your MySQL container.
docker exec -it mydb sh
6. Let’s check the version number by running the same command again, only this time from within the new shell session in the container.
mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version
7. Type exit to leave the interactive shell session.
exit

Task 2: Package and run a custom app using Docker
In this step you’ll learn how to package your own apps as Docker images using a Dockerfile.
The Dockerfile syntax is straightforward. In this task, we’re going to create a simple NGINX website from a Dockerfile.
Build a simple website image
Let’s have a look at the Dockerfile we’ll be using, which builds a simple website that allows you to send a tweet.
1. Make sure you’re in the linux_tweet_app directory.
cd ~/linux_tweet_app
2. Display the contents of the Dockerfile.
cat Dockerfile
Let’s see what each of these lines in the Dockerfile do.
FROM specifies the base image to use as the starting point for this new image you’re creating. For this example we’re starting from nginx:latest.
COPY copies files from the Docker host into the image, at a known location. In this example, COPY is used to copy two files into the image: index.html. and a graphic that will be used on our webpage.
EXPOSE documents which ports the application uses.
CMD specifies what command to run when a container is started from the image. Notice that we can specify the command, as well as run-time arguments.
3. In order to make the following commands more copy/paste friendly, export an environment variable containing your DockerID (if you don’t have a DockerID you can get one for free via Docker Hub).
You will have to manually type this command as it requires your unique DockerID.
export DOCKERID=<your docker id>
4. Echo the value of the variable back to the terminal to ensure it was stored correctly.
echo $DOCKERID
5. Use the docker image build command to create a new Docker image using the instructions in the Dockerfile.
--tag allows us to give the image a custom name. In this case it’s comprised of our DockerID, the application name, and a version. Having the Docker ID attached to the name will allow us to store it on Docker Hub in a later step
. tells Docker to use the current directory as the build context
Be sure to include period (.) at the end of the command.
 docker image build --tag $DOCKERID/linux_tweet_app:1.0 .
6. Use the docker container run command to start a new container from the image you created.
As this container will be running an NGINX web server, we’ll use the --publish flag to publish port 80 inside the container onto port 80 on the host. This will allow traffic coming in to the Docker host on port 80 to be directed to port 80 in the container. The format of the --publish flag is host_port:container_port.
docker container run \
 --detach \
 --publish 80:80 \
 --name linux_tweet_app \
 $DOCKERID/linux_tweet_app:1.0
7. Click here to load the website which should be running.
8. Once you’ve accessed your website, shut it down and remove it.
 docker container rm --force linux_tweet_app

Task 3: Modify a running website
When you’re actively working on an application it is inconvenient to have to stop the container, rebuild the image, and run a new version every time you make a change to your source code.
One way to streamline this process is to mount the source code directory on the local machine into the running container. This will allow any changes made to the files on the host to be immediately reflected in the container.
We do this using something called a bind mount.
When you use a bind mount, a file or directory on the host machine is mounted into a container running on the same host.
Start our web app with a bind mount
1. Let’s start the web app and mount the current directory into the container.
In this example we’ll use the --mount flag to mount the current directory on the host into /usr/share/nginx/html inside the container.
Be sure to run this command from within the linux_tweet_app directory on your Docker host.
 docker container run \
 --detach \
 --publish 80:80 \
 --name linux_tweet_app \
 --mount type=bind,source="$(pwd)",target=/usr/share/nginx/html \
 $DOCKERID/linux_tweet_app:1.0
2. The website should be running.

Modify the running website
Bind mounts mean that any changes made to the local file system are immediately reflected in the running container.
1. Copy a new index.html into the container.
The Git repo that you pulled earlier contains several different versions of an index.html file. You can manually run an ls command from within the ~/linux_tweet_app directory to see a list of them. In this step we’ll replace index.html with index-new.html.
cp index-new.html index.html
2. Go to the running website and refresh the page. Notice that the site has changed.

Even though we’ve modified the index.html local filesystem and seen it reflected in the running container, we’ve not actually changed the Docker image that the container was started from.
To show this, stop the current container and re-run the 1.0 image without a bind mount.
1. Stop and remove the currently running container.
 docker rm --force linux_tweet_app
2. Rerun the current version without a bind mount.
 docker container run \
 --detach \
 --publish 80:80 \
 --name linux_tweet_app \
 $DOCKERID/linux_tweet_app:1.0
3. Notice the website is back to the original version.
4. Stop and remove the current container
docker rm --force linux_tweet_app

Update the image
To persist the changes you made to the index.html file into the image, you need to build a new version of the image.
1. Build a new image and tag it as 2.0
Remember that you previously modified the index.html file on the Docker hosts local filesystem. This means that running another docker image build command will build a new image with the updated index.html
Be sure to include the period (.) at the end of the command.
 docker image build --tag $DOCKERID/linux_tweet_app:2.0 .
2. Let’s look at the images on the system.
docker image ls

Test the new version
1. Run a new container from the new version of the image.
 docker container run \
 --detach \
 --publish 80:80 \
 --name linux_tweet_app \
 $DOCKERID/linux_tweet_app:2.0
2. Check the new version of the website (You may need to refresh your browser to get the new version to load).
The web page will have an orange background.
We can run both versions side by side. The only thing we need to be aware of is that we cannot have two containers using port 80 on the same host.
As we’re already using port 80 for the container running from the 2.0 version of the image, we will start a new container and publish it on port 8080. Additionally, we need to give our container a unique name (old_linux_tweet_app)
3. Run another new container, this time from the old version of the image.
Notice that this command maps the new container to port 8080 on the host. This is because two containers cannot map to the same port on a single Docker host.
 docker container run \
 --detach \
 --publish 8080:80 \
 --name old_linux_tweet_app \
 $DOCKERID/linux_tweet_app:1.0
4. View the old version of the website.

Push your images to Docker Hub
1. List the images on your Docker host.
 docker image ls -f reference="$DOCKERID/*"
2. Before you can push your images, you will need to log into Docker Hub.
 docker login
3. Push version 1.0 of your web app using docker image push.
 docker image push $DOCKERID/linux_tweet_app:1.0
4. Now push version 2.0.
 docker image push $DOCKERID/linux_tweet_app:2.0