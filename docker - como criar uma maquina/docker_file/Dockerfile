# Cada um desses termos, forma uma pilha de camadas

# FROM        == Imagem base
# WORKDIR     == Diretório de trabalho dentro do container
# COPY        == Copia coisas de fora para dentro da imagem
# ADD         == Copia coisas de fora para dentro da imagem: .tar -> copia apenas o que estiver dentro do .tar && arquvo remoto, copia também.  
# RUN         == Comandos a se executar dentro da imagem, durante a execução do container. Cada RUN é uma camada.   
# ENV         == Variáveis de ambiente
# CMD         == Qual comando será executado ao aplicar RUN
# LABEL       == Descrição referente a: chave=valor.  
# USER        == Usuário específico
# VOLUME      == Declaração de (local do) volume (tipo volume).  
# EXPOSE      == Esponha numa porta.  
# ENTRYPOINT  == Principal processo do container. Se morrer, máquina morre junto. Similar ao init, dentro do Linux.  

# Container
# imagem pronta (Read Only)
# imagem pronta, meu container em cima (Read and Write)

# Matar o Container (Overlay File System)
# Kill Read and Write 

FROM python:3.8.12-bullseye
COPY . /app
RUN pip install xlrd and pip install Pillow
WORKDIR /app
CMD python gerador_imagens.py


# Tudo dento de WORKDIR

FROM python:3.8.12-bullseye
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
EXPOSE 5000
ENV FLASK_ENV development
COPY app.py .
CMD ["flask", "run", "--host", "0.0.0.0"]
