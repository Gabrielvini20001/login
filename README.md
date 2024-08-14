from flask import Flask , render_template , request, url_for, redirect
from flask_sqlalchemy import SQLAlchemy 
from flask_login import  LoginManager, UserMixin, login_user, login_required, logout_user, current_user
from werkzeug.security import generate_password_hash,check_password_hash

#configurações inciais framework
app = Flask(__name__)
login_manager=LoginManager()
login_manager.init_app(app)
app.config['SQLALCHEMY_DATABASE_URI']= 'mysql://root:123456@localhost/user'
app.config['SECRET_KEY']='ASSDDFQ1'
db=SQLAlchemy(app)

#classe representando os dados da tabela no banco
class Userr(db.Model, UserMixin):
  __tablename__='usuario2'
  
  id=db.Column(db.Integer, autoincrement=True, primary_key=True)
  nome=db.Column(db.String(20), nullable=False,unique=True )
  senha=db.Column(db.String(200), nullable=False,unique=True )

#criptografando senha
  def __init__(self,nome,senha):
    self.nome=nome
    self.senha=generate_password_hash(senha)
  
  def verify_password(self, pwd):
    return check_password_hash(self.senha, pwd)
with app.app_context():
  db.create_all()

#identificador de usuario
@login_manager.user_loader
def load_user(user_id):
 return Userr.query.get((user_id))

#representação da pagina 
@app.route('/home', methods=['GET', 'POST'])
@login_required
def home():
  return render_template('home.html')

@app.route('/regis', methods=['GET', 'POST'])
def regis():
  if request.method=='POST':
     nome=request.form['username'] 
     pwd=request.form['password'] 
     ne=Userr(nome,pwd)
     db.session.add(ne)
     db.session.commit()
     return redirect(url_for('login'))
 
  return render_template('regis.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
   if request.method=='POST':
     nome=request.form['username']
     pwd=request.form['password']
     se=Userr.query.filter_by(nome=nome).first()
     
     if  se and se.verify_password(pwd):
      login_user(se)
      return redirect(url_for('home'))
   return render_template('login.html')


if __name__=="__main__":
  app.run(debug=True, port=5002)
