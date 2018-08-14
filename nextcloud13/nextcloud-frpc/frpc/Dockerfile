FROM alpine:latest

ENV FRP_VER 0.20.0
ENV FRP_FULL_VER frp_${FRP_VER}_linux_amd64

RUN cd /root \ 
    && wget https://github.com/fatedier/frp/releases/download/v${FRP_VER}/${FRP_FULL_VER}.tar.gz \
    && tar vxzf ${FRP_FULL_VER}.tar.gz \
    && rm ${FRP_FULL_VER}.tar.gz \
    && mv ${FRP_FULL_VER} frpc \
    && rm -rf ./frpc/frps* ./frpc/frpc.ini \
    && mkdir -p /var/log/frpc

COPY frpc.ini /root/frpc/frpc.ini

WORKDIR /root/frpc

CMD [ "./frpc", "-c", "./frpc.ini" ]