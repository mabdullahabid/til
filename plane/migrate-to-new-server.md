# Migrate plane.so deployment to a new server

Create a database dump

    docker exec -e PGPASSWORD=[postgres-password] -t plane-app-plane-db-1 pg_dumpall -c -U plane > dump_$(date +%d-%m-%Y"_"%H_%M_%S).sql

Copy from old server to local

    scp [old-server-url]:/home/ubuntu/plane-selfhost/plane-app/dump_08-05-2024_06_48_21.sql plane_aws_dump_08-05-2024_06_48_21.sql  

Copy from local to new server

    scp plane_aws_dump_08-05-2024_06_48_21.sql [new-server-url]:/root/backups/plane_aws_dump_08-05-2024_06_48_21.sql

Delete and Create Database
    docker compose exec plane-db bash
    dropdb plane -U plane
    createdb plane -U plane

Restore the database dump

    cat plane_aws_dump_08-05-2024_06_48_21.sql | docker exec -e PGPASSWORD=[postgres-password] -i plane-app-plane-db-1 psql -U plane
