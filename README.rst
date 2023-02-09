postgres14 docker stack
=======================

The stack is for local testing: it provides configuration similar to
GCP Cloud SQL running on a db-f1-micro instance.

Example::

    % brew install --cask docker
    % docker swarm init
    % brew install easy-rsa libpq
    % easyrsa init-pki
    % easyrsa build-ca nopass
    % easyrsa build-server-full postgres14 nopass
    % easyrsa build-client-full client nopass
    % mkdir -p ~/.env/postgres14
    % chmod -R 700 ~/.env
    % cat << EOF > ~/.env/postgres14/local.env
    DB_HOST=127.0.0.1
    DB_PORT=5432

    DB_ROOT_USER=postgres
    DB_ROOT_PASSWORD=<password>
    DB_ROOT_DATABASE=postgres

    CA=/opt/homebrew/etc/pki/ca.crt
    DB_SERVER_CERT=/opt/homebrew/etc/pki/issued/postgres14.crt
    DB_SERVER_KEY=/opt/homebrew/etc/pki/private/postgres14.key
    DB_CLIENT_CERT=/opt/homebrew/etc/pki/issued/client.crt
    DB_CLIENT_KEY=/opt/homebrew/etc/pki/private/client.key
    EOF
    % chmod 600 ~/.env/postgres14/local.env
    % set -a
    % source ~/.env/postgres14/local.env
    % echo $DB_ROOT_PASSWORD | \
        docker secret create -l app=postgres14 postgres14-password -
    % docker secret create -l app=postgres14 ca $CA
    % docker secret create -l app=postgres14 postgres14-crt $DB_SERVER_CERT
    % docker secret create -l app=postgres14 postgres14-privkey $DB_SERVER_KEY
    % docker stack deploy -c postgres-stack.yaml postgres14
    % PGPASSWORD=${DB_ROOT_PASSWORD} psql \
        "sslmode=verify-ca \
        sslrootcert=${CA} \
        sslcert=${DB_CLIENT_CERT} \
        sslkey=${DB_CLIENT_KEY} \
        hostaddr=${DB_HOST} \
        user=${DB_ROOT_USER} \
        dbname=${DB_ROOT_DATABASE} \
        port=${DB_PORT}"
    % docker run -it --rm -v postgres14_tmp:/tmp/socket:rw \
        -e PGPASSWORD=${DB_ROOT_PASSWORD} \
        postgres:14.4 \
        psql -h /tmp/socket -U ${DB_ROOT_USER} -d ${DB_ROOT_DATABASE}

