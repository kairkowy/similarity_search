오라클 - Ollama 환경 RAG 예제 시나리오 참고 자료

1. 환경
- 클라이언트 
  - oracle instance client 2.0.1
  - jupyter notebook
  - ptthon 3.12.8 
  - pip 25.0.1
  - library : oml
  - oml4py client
- DBMS
  - oracle23.7
  - oml4py server
- AI Platform 
  - Ollama
- Model
  - LLM : llama3.2
  - 임베딩 : jeffh/intfloat-multilingual-e5-large-instruct:f16

2. jupyter 및 PIP 설치 : 세부내용은 인터넷 참고

Jupyter 다운로드 & 설치
pip upgrade 

jupyter 패스워드 생성

```shell
[dev01@localhost .jupyter]$ ipython
Python 3.12.9 (main, Mar 26 2025, 10:43:27) [GCC 8.5.0 20210514 (Red Hat 8.5.0-24.0.1)]
Type 'copyright', 'credits' or 'license' for more information
IPython 9.0.2 -- An enhanced Interactive Python. Type '?' for help.
Tip: You can find how to type a latex symbol by back completing it `\θ<tab>` will expand to `\theta`.

In [1]: from jupyter_server.auth import passwd

In [2]: passwd()
Enter password:
Verify password:
Out[2]: 'argon2:$argon2id$v=19$m=10240,t=10,p=8$VtpHsaAwl6tnY4ZPM0WoMQ$dkvW1wGfggoknKoI8rBgyZUUeaNNGmBwjJQJKsAuuf4'

# 패스워드(Out) 복사해서 jupyter config에 복제 사용
```

jupyter lab 구성(외부 접속, 패스워드 등)
```shell
cd /home/dev01/.jupyter

jupyter lab --generate-config

vi jupyter_lab_config.py 
#---------- vi
c.LabServerApp.config_file_name = 'jupyter_lab_config.py'
# c.ServerApp.allow_origin = '*'

#외부접속 허용
c.ServerApp.allow_origin = '*'
#작업경로 설정
c.ServerApp.notebook_dir = '/home/dev01/labs/'
#아이피 설정
c.ServerApp.ip = 'localhost'

#포트 설정
c.ServerApp.port = 8888
#비밀번호 암호키 설정
c.ServerApp.password = u'argon2:$argon2id$v=19$m=10240,t=10,p=8$gysSdrw4UaFLdVFdERH9Aw$yAKSB2uH+YdnCoh+u2gH8xA4AGgWhsqJ6x4iObOpUQE'
#시작시 브라우저 실행여부
c.ServerApp.open_browser = False

#c.LabServerApp.workspaces_dir = '/home/dev01/labs'

#c.LabApp.workspaces_dir = '/home/dev01/labs'

c.ServerApp.allow_remote_access = True
#------------ end vi
```
jupyter lab 실행

```shell
nohup jupyter lab --config /home/dev01/.jupyter/jupyter_lab_config.py &
```

3. ollama 구성 : 세부 내용은 인터넷 참고

ollama 다운로드

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

ollama 서비스 수정(외부에서 접속 가능한 서비스 실행)

```config
sudo vi /etc/systemd/system/ollama.service
#-----------------vi
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/home/dev01/python/Python-3.12.9/bin:/home/dev01/oracle/instantclient_23_7:/usr/bin"
Environment="OLLAMA_HOST=0.0.0.0"
User=root

[Install]
#WantedBy=default.target
WantedBy=multi-user.target

#------------ end of vi
```

ollama 서비스 시작

```shell
sudo systemctl daemon-reload
sudo systemctl start ollama
sudo systemctl enable ollama

firewall port(11434) 개방
```
model 다운로드 및 레포지토리 저장

```shell
ollama run llama3.2
ollama run jeffh/intfloat-multilingual-e5-large-instruct:f16
ollama ls
```

Ollama 서비스(임베딩) 확인

```shell
curl -X POST http://192.168.100.7:11434/api/embeddings -d '{"model" : "jeffh/intfloat-multilingual-e5-large-instruct:f16","prompt": "군인공제회 소개해주세요", "stream" : false}'

curl -X POST http://localhost:11434/api/generate -d '{"model" : "llama3.2","prompt": "군인공제회 소개해주세요","stream" : false}'
```

4. DB 환경 준비

- DB 설치 : 233ai free 버전 다운로드

- 23ai free 설치 :

5. 클라이언트 환경 준비

Oracle Instance client 설치

   - OML for python 2-23ai 사용자 가이드 참고(4.5.1.1 Install Oracle Instant Client and the OML4Py Client for Linux)
     https://docs.oracle.com/en/database/oracle/machine-learning/oml4py/2-23ai/mlpug/install-oracle-instant-client-linux-premises-databases.html

OML4PY client 설치
   - OML for python 2-23ai 사용자 가이드 참고(4.5.1.2 Install OML4Py Client for Linux for On-Premises Databases)
     https://docs.oracle.com/en/database/oracle/machine-learning/oml4py/2-23ai/mlpug/install-oml4py-client-linux-premises-databases.html

PIP library for Lab 설치 : 첨부 파일의 requirements4oraclelab.txt 사용

```pip
annotated-types==0.7.0
anyio==4.9.0
argon2-cffi==23.1.0
argon2-cffi-bindings==21.2.0
arrow==1.3.0
asttokens==3.0.0
async-lru==2.0.5
attrs==25.3.0
babel==2.17.0
beautifulsoup4==4.13.3
bleach==6.2.0
certifi==2025.1.31
cffi==1.17.1
charset-normalizer==3.4.1
click==8.1.8
coloredlogs==15.0.1
comm==0.2.2
contourpy==1.3.1
cryptography==44.0.2
cycler==0.12.1
debugpy==1.8.13
decorator==5.2.1
defusedxml==0.7.1
executing==2.2.0
fastjsonschema==2.21.1
filelock==3.18.0
flatbuffers==25.2.10
fonttools==4.56.0
fqdn==1.5.1
fsspec==2025.3.0
gensim==4.3.3
h11==0.14.0
httpcore==1.0.7
httpx==0.28.1
huggingface-hub==0.29.3
humanfriendly==10.0
idna==3.10
ipykernel==6.29.5
ipython==9.0.2
ipython_pygments_lexers==1.1.1
ipywidgets==8.1.5
isoduration==20.11.0
jedi==0.19.2
Jinja2==3.1.6
joblib==1.4.2
jpype1==1.5.2
json5==0.10.0
jsonpatch==1.33
jsonpointer==3.0.0
jsonschema==4.23.0
jsonschema-specifications==2024.10.1
jupyter==1.1.1
jupyter-console==6.6.3
jupyter-events==0.12.0
jupyter-lsp==2.2.5
jupyter_client==8.6.3
jupyter_core==5.7.2
jupyter_server==2.15.0
jupyter_server_terminals==0.5.3
jupyterlab==4.3.6
jupyterlab_pygments==0.3.0
jupyterlab_server==2.27.3
jupyterlab_widgets==3.0.13
kiwisolver==1.4.8
konlpy==0.6.0
langchain-core==0.3.48
langchain-ollama==0.3.0
langsmith==0.3.18
lxml==5.3.1
MarkupSafe==3.0.2
matplotlib==3.8.4
matplotlib-inline==0.1.7
mistune==3.1.3
mpmath==1.3.0
nbclient==0.10.2
nbconvert==7.16.6
nbformat==5.10.4
nest-asyncio==1.6.0
networkx==3.4.2
nltk==3.9.1
notebook==7.3.3
notebook_shim==0.2.4
numpy==2.0.1
ollama==0.4.7
oml @ file:///home/dev01/oracle/oml4py/client/oml-2.1-cp312-cp312-linux_x86_64.whl#sha256=f361fa48170ea492fe83391926b3efca2a8e245aa5e181e34865f53fa1def002
onnx==1.17.0
onnxruntime==1.20.0
onnxruntime_extensions==0.12.0
oracledb==2.4.1
orjson==3.10.16
overrides==7.7.0
packaging==24.2
pandas==2.2.2
pandocfilters==1.5.1
parso==0.8.4
pexpect==4.9.0
pillow==11.1.0
platformdirs==4.3.7
prometheus_client==0.21.1
prompt_toolkit==3.0.50
protobuf==6.30.1
psutil==7.0.0
ptyprocess==0.7.0
pure_eval==0.2.3
pycparser==2.22
pydantic==2.10.6
pydantic_core==2.27.2
Pygments==2.19.1
pyparsing==3.2.3
python-dateutil==2.9.0.post0
python-json-logger==3.3.0
pytz==2025.2
PyYAML==6.0.2
pyzmq==26.3.0
referencing==0.36.2
regex==2024.11.6
requests==2.32.3
requests-toolbelt==1.0.0
rfc3339-validator==0.1.4
rfc3986-validator==0.1.1
rpds-py==0.23.1
safetensors==0.5.3
scikit-learn==1.5.1
scipy==1.14.0
Send2Trash==1.8.3
sentencepiece==0.2.0
setuptools==70.0.0
six==1.17.0
smart-open==7.1.0
sniffio==1.3.1
soupsieve==2.6
stack-data==0.6.3
sympy==1.13.1
tenacity==9.0.0
terminado==0.18.1
threadpoolctl==3.6.0
tinycss2==1.4.0
tokenizers==0.21.1
torch==2.6.0+cpu
tornado==6.4.2
tqdm==4.67.1
traitlets==5.14.3
transformers==4.49.0
types-python-dateutil==2.9.0.20241206
typing_extensions==4.12.2
tzdata==2025.2
uri-template==1.3.0
urllib3==2.3.0
wcwidth==0.2.13
webcolors==24.11.1
webencodings==0.5.1
websocket-client==1.8.0
widgetsnbextension==4.0.13
wrapt==1.17.2
zstandard==0.23.0
```

DB 계정 준비
  - 계정명: vector
  - Role : DB_DEVELOPER_ROLE, create credential, execute on SYS.UTL_HTTP, execute on sys.dbms_network_acl_admin

계정 생성

```sql
CREATE USER vector identified by vector;
ALTER USER vector QUOTA UNLIMITED ON users;

GRANT DB_DEVELOPER_ROLE, create credential to vector;
grant execute on SYS.UTL_HTTP to vector

grant execute on sys.dbms_network_acl_admin to vector;
```

OLLAMA - 오라클 연결 및 테스트

```sql
sqlplus vector/vector@freepdb1

# ACL 생성
BEGIN
  DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
    host => '*',
    ace => xs$ace_type(privilege_list => xs$name_list('connect'),
                       principal_name => 'vector',
                       principal_type => xs_acl.ptype_db));
END;
/
# 오라클 유틸리티 이용한 임베딩 테스트

var embed_ollama_params clob;
exec :embed_ollama_params := '{"provider": "ollama","host":"local","url": "http://180.68.194.221:11434/api/embeddings","model":"llama3.2"}';

select dbms_vector.utl_to_embedding('국방동원정보체계 보안강화 사업', json(:embed_ollama_params)) ollama_output;

OLLAMA_OUTPUT
--------------------------------------------------------------------------------
[-2.1026895E+000,1.87008286E+000,-8.44160765E-002,-1.41829121E+000,


var gent_ollama_params clob;
exec :gent_ollama_params := '{"provider": "ollama","host":"local","url": "http://localhost:11434/api/generate","model":"llama3.2"}';

select dbms_vector.utl_to_generate_text('군인공제회 소개해주세요', json(:gent_ollama_params)) ollama_output;

```

5. 데모 데이터 준비

테이블 생성 : XF_DOC

```sql
CREATE TABLE IF NOT EXISTS XF_DOC(
DOC_ID VARCHAR2(34), 
YEAR VARCHAR2(4), 
EXECUTE_DATE CHAR(14), 
DOC_NUM_ST VARCHAR2(300), 
TITLE VARCHAR2(400), DRAFTER_ID VARCHAR2(40),
DRAFTER_NAME VARCHAR2(30), 
CASTING_VOTER_ID VARCHAR2(40), 
CASTING_VOTER_NAME VARCHAR2(40), 
OWN_DEPT_CODE CHAR(7), 
OWN_DEPT_NAME VARCHAR2(300), 
VECTOR_SOURCE VECTOR, 
PRIMARY KEY(DOC_ID));
```

데이터 로딩 : sqlloader 사용

  - control 파일 생성 

```shelll
 
options (skip=1)
load data
infile './data/xf_doc.csv'
  into table xf_doc
  fields terminated by ','
  (
  DOC_ID CHAR(34), 
  YEAR CHAR(4), 
  EXECUTE_DATE CHAR, 
  DOC_NUM_ST CHAR, 
  TITLE CHAR, 
  DRAFTER_ID CHAR,
  DRAFTER_NAME CHAR, 
  CASTING_VOTER_ID CHAR, 
  CASTING_VOTER_NAME CHAR, 
  OWN_DEPT_CODE CHAR, 
  OWN_DEPT_NAME CHAR, 
  );
```

  - sqlloader 실행

```shell
sqlldr vector/vector@freepdb1 control=xf_doc.ctl 

```
  - 테이블 정보 확인
``` sql

analyze table xf_doc compute statistics;

select table_name, num_rows, blocks from user_tables where table_name = 'XF_DOC';

TABLE_NAME                       NUM_ROWS     BLOCKS
------------------------------ ---------- ----------
XF_DOC                                285         13


select a.table_name, b.bytes/1048576 MB from user_tab_columns a, user_segments b 
where a.table_name = b.segment_name and a.table_name = 'XF_DOC' 
and a.data_type = 'VECTOR'

TABLE_NAME                             MB
------------------------------ ----------
XF_DOC                               .125

```