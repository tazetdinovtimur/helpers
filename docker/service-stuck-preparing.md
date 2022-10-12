# Docker swarm service stuck in preparing state (mac)
Docker swarm service has been created, but stuck in preparing state and there is nothing in logs.
This could happen when there are two images with the same name and one of the tags is \<none\>:

    REPOSITORY                       TAG             IMAGE ID       CREATED         SIZE
    repository/image                 4.8.4           a61d7666ccb5   6 weeks ago     4.95GB
    repository/image                 <none>          1711fdf8b852   4 months ago    4.92GB

Deleting the spare image fixed the problem and docker swarm was able to start a container