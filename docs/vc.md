# Video conferencing

eduMEET is an Open Source web-conferencing service from GÉANT portfolio. It is
an interactive, web-based videoconferencing platform, supporting distant
learning processes, including audio-video communication, chat, screen and file
sharing.

See [edumeet.org](https://edumeet.org/).

## Software Architecture

The webserver serving the webpage being a frontpage of the eduMEET conferencing
software is build on top of Node.js. It uses Redis storage to cache the
sessions and mediasoup WebRTC to provide the infrastructure to efficiently
receive and send video and audio. Both media channels are sent via UDP protocol
and are automatically adjusted in order to provide reliable and real time
communication protocol. 

## Deployment

The deployment of the application is well documented on the product [web
page](https://edumeet.org/build/). Administrator have two choices of
installation, one is based on Ansible, second directly on deploying Docker
images. The former is easier to maintain the latter easier for testing and
development. 

It is also quite vital to have a configured instance of a TURN server. One can
either install a dedicated instance of coturn server, or register under the
[eduturn.org](https://eduturn.org/).

## Scaling up

There is no straightforward path for horizontal scaling up of an instance. One
can follow the guidelines
[here](https://github.com/edumeet/edumeet/wiki/Scaling-and-recommended-Hardware)
and [here](https://github.com/edumeet/edumeet/blob/master/HAproxy.md).

## Notes

The crucial part of the VC server is a reliable, high-bandwidth network
connection. Without extensive modification of configuration it requires opened
a defined set of ports, including TCP 80, 443, and a UDP range 40000:49999.
The eduMEET relies on undisturbed UDP connection with all of its
clients. During our experience with this software, we have observed that
firewalls, particularly with enabled IDP (Intrusion Detection and Prevention),
may interrupt this process. 

Additionally, having a machine with no public IP address assigned directly to
the machine, might introduce extra difficulties in configuring properly the
server and therefore should be avoided.

Basic styling of the application, like for example changing logo, or colors can
be done by editing the the application configuration. Example values and
cofiguration can be found in
[the eduMEET repository](https://github.com/edumeet/edumeet/blob/master/app/public/config/config.example.js).

