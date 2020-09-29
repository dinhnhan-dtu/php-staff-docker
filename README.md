# GUIDE
docker build -t php-staff -f docker-context/php-staff/Dockerfile docker-context/php-staff

docker-compose up -d

# Init .env
docker run -it --rm \
-v ~/projects/vnlab/point/staff:/tmp/point \
-u nhandd \
-w /tmp/point \
php-staff php init --env=local --overwrite=All

# /etc/hosts

127.0.0.1 php-staff.local

# URL : https://php-staff.local:10443