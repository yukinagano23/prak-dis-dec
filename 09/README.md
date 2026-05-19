# Modul 9
## Arsitektur Microservices untuk Sistem Terdistribusi: GraphQL dan gRPC

## Langkah-Langkah Praktikum Modul 9 di Windows
### Prasyarat
Pastikan sudah terinstall:
1. uv → download dari https://docs.astral.sh/uv/getting-started/installation/ (pakai PowerShell: powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex")
2. Python (akan diinstall via uv)
3. Git Bash atau PowerShell sebagai terminal

---

### 0. Setup Awal
#### Install & Pin Python 3.14
```bash
uv python pin cpython-3.14.5-windows-x86_64
uv python list
```
<img width="840" height="382" alt="image" src="https://github.com/user-attachments/assets/5feb57f8-4d18-4229-86b3-c69d60fe3649" />

#### Struktur Direktori
Buat struktur folder berikut secara manual atau lewat terminal:
```bash
src/
├── graphql/
│   ├── client/
│   └── server/
└── grpc/
    ├── client/
    └── server/
```

---

### 1. GraphQL
#### 1.1. Setup GraphQL Server
Masuk ke direktori server GraphQL:
```bash
cd graphql
cd server
```
<img width="840" height="43" alt="image" src="https://github.com/user-attachments/assets/a708199a-0105-448d-8a4b-ba0a59c50829" />

Buat virtual environment:
```bash
uv venv
```
<img width="843" height="73" alt="image" src="https://github.com/user-attachments/assets/8e7afd38-e51a-45bc-a121-6519ec901d16" />

Aktifkan venv di PowerShell:
```bash
.venv\Scripts\Activate.ps1
```
<img width="840" height="202" alt="image" src="https://github.com/user-attachments/assets/ad62214c-3ca8-49c5-8ae7-daa8329fb6ba" />

#### Masalah: Execution Policy diblokir
PowerShell melarang menjalankan script .ps1 karena kebijakan keamanan default Windows (running scripts is disabled on this system).
#### Solusi
Jalankan perintah ini dulu sekali saja:
```bash
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
Ketik Y lalu Enter ketika diminta konfirmasi.
Setelah itu, aktifkan venv seperti biasa:
```bash
.venv\Scripts\Activate.ps1
```
<img width="845" height="127" alt="image" src="https://github.com/user-attachments/assets/eaa1e6af-c8dc-4dc3-b1f1-dabf709d1552" />

Buat file requirements.txt dengan perintah:
```bash
notepad requirements.txt
```
setelah notepad terbuka, isikan:
```bash
strawberry-graphql[debug-server]
sqlmodel
```
<img width="739" height="201" alt="image" src="https://github.com/user-attachments/assets/909b7fe3-084b-40eb-81b0-4ff446547a2a" />

Install paket:
```bash
uv pip install -r requirements.txt
```
<img width="846" height="287" alt="image" src="https://github.com/user-attachments/assets/29f1d47a-17e6-4f4c-a08b-1a7e4ff587a8" />

Salin file departemen-sdm.db ke direktori ini (dari modul 8).

Buat file graphql-server.py:
```bash
import typing
import strawberry
from sqlmodel import Field, Session, SQLModel, create_engine, select

@strawberry.type
class Sdm(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    npp: str
    nama: str

engine = create_engine("sqlite:///departemen-sdm.db")

def get_sdms():
    with Session(engine) as session:
        statement = select(Sdm)
        results = session.exec(statement).all()
    return results

@strawberry.type
class Query:
    sdms: typing.List[Sdm] = strawberry.field(resolver=get_sdms)

schema = strawberry.Schema(query=Query)
```
<img width="1081" height="487" alt="image" src="https://github.com/user-attachments/assets/284a8e54-e56e-40f8-8eb1-3cf8a0a31242" />

Jalankan server:

```bash
strawberry dev graphql-server
```
<img width="845" height="41" alt="image" src="https://github.com/user-attachments/assets/2b3bf30f-519d-4f11-b566-99d53d87a61a" />
<img width="1595" height="816" alt="image" src="https://github.com/user-attachments/assets/e4c0437a-c633-4e9a-81aa-20548881e9b0" />

Server berjalan di http://localhost:8000/graphql

#### 1.2. Setup GraphQL Client
Buka terminal baru, masuk ke direktori client:
```bash
cd src\graphql\client
```
<img width="929" height="153" alt="image" src="https://github.com/user-attachments/assets/0e5f9ef1-109e-4634-ae07-1413f56c1477" />

Deactivate venv lama jika masih aktif:
```bash
deactivate
```

Buat venv baru:
```bash
uv venv
.venv\Scripts\Activate.ps1
```
<img width="930" height="137" alt="image" src="https://github.com/user-attachments/assets/9c2461a1-4788-4a08-a69b-62c08a22b073" />

Buat requirements.txt:
```bash
gql[aiohttp]
```
<img width="1074" height="192" alt="image" src="https://github.com/user-attachments/assets/e6f67327-fac6-441d-8ecc-a532b0634b93" />

Install paket:
```bash
uv pip install -r requirements.txt
```
<img width="932" height="287" alt="image" src="https://github.com/user-attachments/assets/4d64c6d2-4bb5-4ffe-a53a-7982051f3ead" />

Buat file graphql-client.py:
```bash
import asyncio
from gql import Client, gql
from gql.transport.aiohttp import AIOHTTPTransport

async def main():
    transport = AIOHTTPTransport(url="http://localhost:8000/graphql")
    client = Client(transport=transport)

    query = gql("""
        query getSdms {
            sdms {
                id
                npp
                nama
            }
        }
    """)

    async with client as session:
        result = await session.execute(query)
        print(result)

asyncio.run(main())
```
<img width="1077" height="471" alt="image" src="https://github.com/user-attachments/assets/7c0c7b40-b6a5-4c2e-abc1-c16bb20b8bad" />

Jalankan client (pastikan server masih berjalan):
```bash
python graphql-client.py
```
<img width="929" height="104" alt="image" src="https://github.com/user-attachments/assets/9cc04654-4edb-4225-a860-230955d633d9" />

---

### 2. gRPC
#### 2.1. Setup gRPC Server
Buka terminal baru, masuk ke direktori server gRPC:
```bash
cd src\grpc\server
```
Buat venv:
```bash
uv venv
.venv\Scripts\Activate.ps1
```
<img width="842" height="132" alt="image" src="https://github.com/user-attachments/assets/903474ae-a903-4e96-9b1d-f19349357dfd" />

Buat requirements.txt:
```bash
grpcio
grpcio-tools
sqlmodel
```
<img width="1077" height="225" alt="image" src="https://github.com/user-attachments/assets/925947ac-7cf7-46c6-8c66-2fe665f1ca21" />

Install paket:
```bash
uv pip install -r requirements.txt
```
<img width="838" height="288" alt="image" src="https://github.com/user-attachments/assets/3a624b47-7a15-4153-aaac-7c2de5c54110" />

Salin departemen-sdm.db ke direktori ini.

Buat file service.proto:
```bash
syntax = "proto3";

package mydb;

message SdmRequest {
    int32 id = 1;
}

message SdmResponse {
    int32 id = 1;
    string npp = 2;
    string nama = 3;
}

service SdmService {
    rpc GetSdm (SdmRequest) returns (SdmResponse);
}
```
<img width="1077" height="377" alt="image" src="https://github.com/user-attachments/assets/e5033add-601c-4584-ac91-865643be8d15" />

Kompilasi file proto:
```bash
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. service.proto
```
<img width="844" height="75" alt="image" src="https://github.com/user-attachments/assets/3b7bc6b6-35cf-4c09-ab3e-8d40613f6269" />

Buat file server.py:
```bash
import grpc
from concurrent import futures
from sqlmodel import Field, Session, SQLModel, create_engine, select
import service_pb2
import service_pb2_grpc

class Sdm(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    npp: str
    nama: str

engine = create_engine("sqlite:///departemen-sdm.db")

class SdmServiceServicer(service_pb2_grpc.SdmServiceServicer):
    def GetSdm(self, request, context):
        with Session(engine) as session:
            statement = select(Sdm).where(Sdm.id == request.id)
            results = session.exec(statement)
            sdm_result = results.first()

            if sdm_result is None:
                context.set_code(grpc.StatusCode.NOT_FOUND)
                context.set_details("User not found")
                return service_pb2.SdmResponse()

            return service_pb2.SdmResponse(
                id=sdm_result.id,
                npp=sdm_result.npp,
                nama=sdm_result.nama,
            )

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    service_pb2_grpc.add_SdmServiceServicer_to_server(SdmServiceServicer(), server)
    server.add_insecure_port('[::]:50051')
    print("Server started. Listening on port 50051...")
    server.start()
    server.wait_for_termination()

if __name__ == '__main__':
    serve()
```
<img width="1076" height="808" alt="image" src="https://github.com/user-attachments/assets/be386578-7b61-4c1d-97ab-e9fac1750979" />

Jalankan server:
```bash
python server.py
```

#### 2.2. Setup gRPC Client
Buka terminal baru, masuk ke direktori client gRPC:
```bash
cd src\grpc\client
```
<img width="841" height="232" alt="image" src="https://github.com/user-attachments/assets/9d6d3401-d648-4b9d-91b5-75223c46b578" />

Buat venv:
```bash
uv venv
.venv\Scripts\Activate.ps1
```
<img width="844" height="134" alt="image" src="https://github.com/user-attachments/assets/f52d7a50-c62f-4a29-ab74-6216118c79f9" />

Buat requirements.txt:
```bash
grpcio
grpcio-tools
```
<img width="1081" height="196" alt="image" src="https://github.com/user-attachments/assets/f0ecbda6-0a9f-432e-8652-1e5af3e7a474" />

Install paket:
```bash
uv pip install -r requirements.txt
```
<img width="844" height="155" alt="image" src="https://github.com/user-attachments/assets/79ee6f08-c2af-4224-b456-5ac10def83b6" />

Copy file service.proto, service_pb2.py, dan service_pb2_grpc.py dari direktori server ke direktori client ini:

Buat file client.py:
```bash
import grpc
import service_pb2
import service_pb2_grpc

def run_client():
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = service_pb2_grpc.SdmServiceStub(channel)

        # Request dengan id yang ada
        request = service_pb2.SdmRequest(id=1)
        try:
            response = stub.GetSdm(request)
            print(f"Sdm Name: {response.nama}")
        except grpc.RpcError as e:
            print(f"Error: {e.code()} - {e.details()}")

        # Request dengan id yang tidak ada
        request_not_found = service_pb2.SdmRequest(id=10000)
        try:
            response = stub.GetSdm(request_not_found)
            print(f"Sdm Name: {response.nama}")
        except grpc.RpcError as e:
            print(f"Error: {e.code()} - {e.details()}")

if __name__ == '__main__':
    run_client()
```
<img width="1072" height="554" alt="image" src="https://github.com/user-attachments/assets/06c59b76-cccb-42bd-ab93-e959e76862f5" />

Jalankan client (pastikan gRPC server masih berjalan):
```bash
python client.py
```
Output yang diharapkan:
```bash
Sdm Name: Karyawan 1
Error: StatusCode.NOT_FOUND - User not found
```
<img width="839" height="106" alt="image" src="https://github.com/user-attachments/assets/89794f0e-0e47-4f84-9b73-2ee715cf7bcd" />









