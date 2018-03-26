FROM mongo

RUN mkdir -p /data/db

EXPOSE 27017

CMD mongod --smallfiles --logpath=/dev/null  --quiet
