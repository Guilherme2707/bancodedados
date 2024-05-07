import sqlalchemy
from sqlalchemy.orm import declarative_base, relationship, Session
from sqlalchemy import Column, Integer, String, ForeignKey, create_engine, inspect, select, func

Base = declarative_base()

class User(Base):
     __tablename__ = "user_account"
     # Atributos
     id = Column(Integer, primary_key=True) 
     name = Column(String)
     fullname = Column(String) 
     address = relationship("Address", back_populates="user", cascade="all, delete-orphan")

     def __repr__(self):
          return f"id={self.id}, name={self.name}, fullname={self.fullname}"

class Address(Base):
     __tablename__ = "address"
     id = Column(Integer, primary_key=True)   
     email_address = Column(String(50))
     user_id = Column(Integer, ForeignKey("user_account.id"))
     user = relationship("User", back_populates="address")

     def __repr__(self):
          return f"Address(id={self.id}, email={self.email_address}, user_id={self.user_id})"

print(User.name)


#Conexão Com o Banco de Dados
engine = create_engine("sqlite://")

#Criando as Classes como tabelas no banco de dados
Base.metadata.create_all(engine)

#Investiga o Esquema no banco de dados
inspect_engine = inspect(engine)
print(inspect_engine.has_table("user_account"))

print(inspect_engine.get_table_names())

with Session(engine) as session:
    guilherme = User(
        name='guilherme',
        fullname='Guilherme Oliveira',
        address=[Address(email_address='guilhermealmeida2607@gmail.com')]   
     )

    nicoly = User(
        name='nicoly',
        fullname='Nicoly Rocha',
        address=[Address(email_address='nicolyrocha@gmail.com'),
                 Address(email_address='nicolyrocha2@gmail.org')]   
    ) 

#Enviando as infromações para o banco de dados
session.add_all([guilherme,nicoly])   
session.commit()

stmt = select(User).where(User.name.in_(['guilherme','nicoly']))
print("Recuperando Usuaria apartir de Filtragem: \n")
for user in session.scalars(stmt):
     print(user)

print("Recuperando Os emails de NICOLY: \n")
stmt_address = select(Address).where(Address.user_id.in_([2]))
for address in session.scalars(stmt_address):
     print(address)  

order = select(User).order_by(User.fullname.asc()) 
print("\n""Recuperando Informação de maneira ordenada")
for result in session.scalars(order):
     print(result)  

stmt_join = select(User.fullname, Address.email_address).join_from(Address,User)
for result in session.scalars(stmt_join):
     print(result)
print(select(User.fullname, Address.email_address).join_from(Address,User))     

connection = engine.connect()
results = connection.execute(stmt_join).fetchall()
print("Executando Apartir da conexão!!")
for result in results:
     print(result)

stmt_count = select(func.count()).select_from(User)
for result in session.scalars (stmt_count):
     print ("Número de Users: ", result)
