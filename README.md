# Entrega-1-
Primeira entrega Bonato

package br.com.usjt.model;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.io.LineNumberReader;
import java.util.ArrayList;
import java.util.Locale;
import java.util.Observable;
import java.util.ResourceBundle;

import javax.swing.JOptionPane;

import br.com.usjt.view.TelaEntrarComCodigo;
import br.com.usjt.view.TelaGerarCodigo;

public class Acesso  extends Observable{

	private int agencia, conta, senha, codigoDeAcesso;
	private FileReader txtConta;
	private boolean validar;
	private ResourceBundle idioma;

	public int getAgencia() {
		return agencia;
	}

	public void setAgencia(int agencia) {
		this.agencia = agencia;
		setChanged();
		notifyObservers();
	}

	public int getConta() {
		return conta;
	}

	public void setConta(int conta) {
		this.conta = conta;
		setChanged();
		notifyObservers();
	}

	public int getSenha() {
		return senha;
	}

	public void setSenha(int senha) {
		this.senha = senha;
		setChanged();
		notifyObservers();
	}

	public int getCodigoDeAcesso() {
		return codigoDeAcesso;		
	}

	public void setCodigoDeAcesso(int codigoDeAcesso) {
		this.codigoDeAcesso = codigoDeAcesso;
		setChanged();
		notifyObservers();
	}

	public ResourceBundle getIdioma() {
		return idioma;
	}

	public void setIdioma(ResourceBundle idioma) {
		this.idioma = idioma;
	}


	// enable user to open file
	public FileReader openFile() throws FileNotFoundException {
		txtConta = new FileReader (""+ getConta());
		return txtConta;
	} // end method openFile
	// read record from file

	public boolean validar() throws IOException{

		BufferedReader entrada = new BufferedReader(openFile());

		ArrayList<Integer> linhasTxt = new ArrayList<Integer>();
		String texto;
		while((texto = entrada.readLine()) != null){

			linhasTxt.add(Integer.parseInt(texto, 16));//converte de hexadecimal para inteiro

		}

		int agencia = linhasTxt.get(1);
		int senha = linhasTxt.get(2);
		if(agencia == getAgencia() && senha == getSenha()){
			validar = true;
			JOptionPane.showMessageDialog(null, "Acesso Autorizado!");

			File arquivoLeitura = new File("" + getConta()); 
			LineNumberReader linhaLeitura = new LineNumberReader(openFile());  
			linhaLeitura.skip(arquivoLeitura.length());  
			int qtdLinha = linhaLeitura.getLineNumber(); 

			if(qtdLinha != 3){
				TelaGerarCodigo telaG = new TelaGerarCodigo();
				telaG.setNumConta(getConta());
				try{//verifica se nenhum idioma foi selecionado
					telaG.internacionalizar(getIdioma());
				}catch(NullPointerException e){//se nenhum idioma for selecionado ele começa com padrão pelo português
					telaG.internacionalizar(ResourceBundle.getBundle("projeto", new Locale("pt", "BR")));
				}
				telaG.setAgencia(getAgencia());
				telaG.setSize(800, 400);
				telaG.setVisible(true);
			}else{
				TelaEntrarComCodigo entCod = new TelaEntrarComCodigo();
				entCod.setNumConta(getConta());
				entCod.setAgencia(getAgencia());
				try{//verifica se nenhum idioma foi selecionado
					entCod.internacionalizar(getIdioma());
				}catch(NullPointerException e){//se nenhum idioma for selecionado ele começa com padrão pelo português
					entCod.internacionalizar(ResourceBundle.getBundle("projeto", new Locale("pt", "BR")));
				}
			
				
			}	
		}else{
			JOptionPane.showMessageDialog(null, "Favor verifique as informações digitadas e tente novamente.");
			validar = false;
		}
		return validar;
	}
}

package br.com.usjt.model;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Observable;

import javax.swing.JOptionPane;

import br.com.usjt.DAO.Conexao;
import br.com.usjt.DAO.ContaDAO;

public class Conta extends Observable{
	private int agencia, saldo, numConta;
	private String nome;

	public Conta() {
		// TODO Auto-generated constructor stub
	}

	public String getNome() {
		return nome;
	}
	
	public void setNome(String nome) {
		this.nome = nome;
	}
	
	public int getAgencia() {		
		return agencia;
	}

	public void setAgencia(int agencia) {
		this.agencia = agencia;
		setChanged();
		notifyObservers();
	}

	public int getNumConta() {
		return numConta;
	}

	public void setNumConta(int numConta) {
		this.numConta = numConta;
		setChanged();
		notifyObservers();

	}

	public int getSaldo() {
		return saldo;
	}

	public void setSaldo(int saldo) {
		this.saldo = saldo;
		setChanged();
		notifyObservers();
	}

	public String consultarSaldo(){

		String saldo = "";
		Connection conn;
		try {
			conn = new Conexao().connection();
			ContaDAO contaDAO = new ContaDAO();
			try{

				ResultSet rs = contaDAO.selec(getNumConta()).executeQuery();

				while(rs.next()){  
					setSaldo(Integer.parseInt(saldo = rs.getString(5)));				 
				}  

				conn.close();

			}catch(Exception e){
				e.printStackTrace();
				try{
					conn.rollback();
				}catch (SQLException e1){
					System.out.print(e1.getStackTrace());
				}
			}finally{
				try {
					if (contaDAO.selec(getNumConta()) != null){
						try{
							contaDAO.selec(getNumConta()).close();
						}catch (SQLException e1){
							System.out.print(e1.getStackTrace());
						}
					}
				} catch (ClassNotFoundException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		} catch (SQLException e2) {
			// TODO Auto-generated catch block
			e2.printStackTrace();
		}
		return saldo;
	}

	public void imprimir() throws IOException{
		SimpleDateFormat formatoData = new SimpleDateFormat("dd-MM-yyyy" + " HH.mm.ss");//cria um tipo date para colocar a data de que foi criado o arquivo no nome do arquivo 
		Calendar data = Calendar.getInstance();

		File arquivo;
		
		String nomeArq = "Saldo da conta " + getNumConta() + " do dia e hora " + formatoData.format(data.getTime()) + ".txt";
		//criando arquivo para preencher com o extrato 
		try
		{
			arquivo = new File(nomeArq);
			arquivo.createNewFile();
			JOptionPane.showMessageDialog(null,"Arquivo '"+ nomeArq + "' criado!","Arquivo",1);
		}
		//mostrando erro em caso se nao for possivel gerar arquivo
		catch(Exception erro){
			JOptionPane.showMessageDialog(null,"Arquivo nao pode ser gerado!","Erro",0);
		}

		FileWriter output = new FileWriter(new File(nomeArq),true);
		PrintWriter gravarArq = new PrintWriter(output, true);//coloca o arquivo na variavel para preenchelo (o true permite que escreve em um arquivo de texto ja preenchido)
	
		String newLine = System.getProperty("line.separator");
	
		String saldoDoDia  = "\t" + "Saldo do dia e hora " + formatoData.format(data.getTime())
		+ newLine + newLine + "Nome: " + getNome()
		+ newLine +"Conta: " + getNumConta()
		+ newLine + "Agencia: " + getAgencia()
		+ newLine + "saldo: " + getSaldo();

		gravarArq.format("\n" + saldoDoDia);
	}
}
package br.com.usjt.model;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.text.ParseException;
import java.util.ArrayList;
import java.util.Date;
import java.util.Observable;

import javax.swing.JOptionPane;

import br.com.usjt.DAO.Conexao;
import br.com.usjt.DAO.DebitoAutomaticoDAO;

public class DebitoAutomatico extends Observable{

	private int codigoConsumidor, codDebito;
	private String operadora, tipoDebito;
	private Date dataDebito;
	private double valorDebito;
	private DebitoAutomaticoDAO debDao;
	private Movimento movimento;
	private int numConta, numAgencia;
	

	public int getCodDebito() {
		return codDebito;
	}

	public void setCodDebito(int codDebito) {
		this.codDebito = codDebito;
	}

	public String getTipoDebito() {
		return tipoDebito;
	}
	public void setTipoDebito(String tipoDebito) {
		this.tipoDebito = tipoDebito;
		setChanged();
		notifyObservers();
	}
	public String getOperadora() {
		return operadora;
	}
	public void setOperadora(String operadora) {
		this.operadora = operadora;
		setChanged();
		notifyObservers();
	}
	public int getCodigoConsumidor() {
		return codigoConsumidor;
	}
	public void setCodigoConsumidor(int codigoConsumidor) {
		this.codigoConsumidor = codigoConsumidor;
		setChanged();
		notifyObservers();
	}
	public Date getDataDebito() {
		return dataDebito;
	}
	public void setDataDebito(Date dataDebito) throws ParseException {    
		this.dataDebito = dataDebito;
	}
	public double getValorDebito() {
		return valorDebito;
	}
	public void setValorDebito(double valorDebito) {
		this.valorDebito = valorDebito;
	}

	public int getNumConta() {
		return numConta;
	}
	
	private int getAgencia() {
		return numAgencia;
	}
	
	public void setNumAgencia(int numAgencia) {
		this.numAgencia = numAgencia;
	}
	
	public void setNumConta(int numConta) {
		this.numConta = numConta;
	}

	public ArrayList<DebitoAutomatico> consultar() throws SQLException, ClassNotFoundException{

		//cria um arrayList para serem armazenado todas as informações do banco de dados
		ArrayList<DebitoAutomatico> resultadoPesquisa = new ArrayList<DebitoAutomatico>();
		Connection conn = new Conexao().connection();

		DebitoAutomaticoDAO debDAO = new DebitoAutomaticoDAO();

		ResultSet rs = null;

		try{
			rs = debDAO.selec(getNumConta()).executeQuery();
			while (rs.next()){
				DebitoAutomatico rsp = new DebitoAutomatico();
				rsp.setCodDebito(rs.getInt(1));
				rsp.setTipoDebito(rs.getString(2));
				rsp.setOperadora(rs.getString(3));
				rsp.setCodigoConsumidor(rs.getInt(4));
				rsp.setDataDebito(rs.getDate(5));  
				rsp.setValorDebito(rs.getDouble(6));
				rsp.setNumConta(rs.getInt(7));
				resultadoPesquisa.add(rsp);
			}

			conn.close();
		}catch (Exception e){
			e.printStackTrace();
			try{
				conn.rollback();
			}catch (SQLException e1){
				System.out.print(e1.getStackTrace());
			}
		}
		return resultadoPesquisa;
	}

	public void gerarDebito() throws SQLException, ClassNotFoundException{
		debDao = new DebitoAutomaticoDAO();

		debDao.insert(getTipoDebito(), getOperadora(), getCodigoConsumidor(), getDataDebito(), getValorDebito(), getNumConta()).execute();

		JOptionPane.showMessageDialog(null, "Debito automatico inserido com sucesso !!");
		
		movimento = new Movimento();
		
		movimento.setValorDaOperacao(getValorDebito());
		movimento.setDataDoMovimento(getDataDebito());
		movimento.geraMovimento(getAgencia(), getNumConta(), 0, 0, "Debito Automatico");			
	}
}

package br.com.usjt.model;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Observable;
import javax.swing.JOptionPane;
import br.com.usjt.DAO.DispenserDAO;

public class Dispenser extends Observable{

	private int nota, quantidade;

	public Dispenser(int nota, int quantidade) {
		this.nota = nota;
		this.quantidade = quantidade;
	}

	public Dispenser() {
	}

	public int getNota() {
		return nota;
	}

	public int getQuantidade() {
		return quantidade;
	}

	public void setNota(int nota) {
		this.nota = nota;
		setChanged();
		notifyObservers();

	}

	public void setQuantidade(int quantidade) {
		this.quantidade = quantidade;
		setChanged();
		notifyObservers();
	}

	public void contarNotas(double valorRetirar){

		try{
			//cria um arrayList para serem armazenado todas as informações do banco de dados
			ArrayList<Dispenser> notas = new ArrayList<Dispenser>();

			//cria um arrayList para serem armazenado todas as informações do banco de dados

			DispenserDAO dispenserDAO = new DispenserDAO();

			ResultSet rs = null;

			rs = dispenserDAO.recuperarNotas().executeQuery();

			while (rs.next()){
				notas.add(new Dispenser(rs.getInt(1), rs.getInt(2)));
			}

			int notas50 = notas.get(2).getQuantidade();
			int notas20 = notas.get(1).getQuantidade();
			int notas10 = notas.get(0).getQuantidade();
			
			if(notas10  == 0 || notas20 == 0 || notas50 == 0){
				JOptionPane.showMessageDialog(null, "As notas disponiveis estao acabando. Favor consulte o administrador "
						+ " do sistema para que ele resete o dispenser");
			}

			notas50 -= valorRetirar / 50;

			if (notas50 < 0) {
				notas50 =  notas.get(2).getQuantidade();
				valorRetirar -= (notas50 * 50);
				notas50 = 0;
			} else {
				valorRetirar %= 50;
			}

			notas20 -= valorRetirar / 20;

			if (notas20 < 0) {
				notas20 = notas.get(1).getQuantidade();
				valorRetirar -= (notas20 * 20);
				notas20 = 0;
			} else {
				valorRetirar %= 20;
			}

			notas10 -= valorRetirar / 10;

			if (notas10 < 0) {
				throw new Exception();
			} else {
				valorRetirar %= 10;
			}

			notas.get(2).setQuantidade(notas50);
			notas.get(1).setQuantidade(notas20);
			notas.get(0).setQuantidade(notas10);

			dispenserDAO.retirarNota(notas);

			JOptionPane.showMessageDialog(null, "Nao emitimos comprovante para essa operacao");
		}catch(Exception e){
		}
	}

	public ArrayList<Dispenser> consultarExtratoDeNotas(){

		//cria um arrayList para serem armazenado todas as informações do banco de dados
		ArrayList<Dispenser> notas = new ArrayList<Dispenser>();

		//cria um arrayList para serem armazenado todas as informações do banco de dados
	
		try {
			DispenserDAO dispenserDAO = new DispenserDAO();
			
			ResultSet rs = null;

			rs = dispenserDAO.recuperarNotas().executeQuery();

			while (rs.next()){
				notas.add(new Dispenser(rs.getInt(1), rs.getInt(2)));
			}	
			
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return notas;
	}
}

package br.com.usjt.model;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Date;
import java.util.Observable;

import br.com.usjt.DAO.LogDAO;

public class Log extends Observable{
	private int codigoItemLog, agencia, conta, codigoCliente, codigoMovimento, agenciaDestino, contaDestino;
	private Date dataOperacao;
	private double valor;
	private String operacao;
	private LogDAO logDAO;

	public int getCodigoItemLog() {
		return codigoItemLog;
	}
	public void setCodigoItemLog(int codigoItemLog) {
		this.codigoItemLog = codigoItemLog;
		setChanged();
		notifyObservers();
	}
	
	public int getAgencia() {
		return agencia;
	}
	public void setAgencia(int agencia) {
		this.agencia = agencia;
		setChanged();
		notifyObservers();
	}
	public int getConta() {
		return conta;
	}
	public void setConta(int conta) {
		this.conta = conta;
		setChanged();
		notifyObservers();
	}
	public Date getDataOperacao() {
		return dataOperacao;
	}
	public void setDataOperacao(Date dataOperacao) {
		this.dataOperacao = dataOperacao;
		setChanged();
		notifyObservers();
	}
	public double getValor() {
		return valor;
	}
	public void setValor(double valor) {
		this.valor = valor;
		setChanged();
		notifyObservers();
	}
	
	public int getCodigoCliente() {
		return codigoCliente;
	}
	
	public void setCodigoCliente(int codigoCliente) {
		this.codigoCliente = codigoCliente;
		setChanged();
		notifyObservers();
	}
	
	public int getCodigoMovimento() {
		return codigoMovimento;
	}
	
	public void setCodigoMovimento(int codMovimento) {
		this.codigoMovimento = codMovimento;
		setChanged();
		notifyObservers();
	}
	
	
	public String getOperacao() {
		return operacao;
	}
	
	public void setOperacao(String operacao) {
		this.operacao = operacao;
		setChanged();
		notifyObservers();
	}
	
	public int getContaDestino() {
		return contaDestino;
	}
	
	public void setContaDestino(int contaDestino) {
		this.contaDestino = contaDestino;
		setChanged();
		notifyObservers();
	}
	
	public int getAgenciaDestino() {
		return agenciaDestino;
	}
	
	public void setAgenciaDestino(int agenciaDestino) {
		this.agenciaDestino = agenciaDestino;
		setChanged();
		notifyObservers();
	}

	public void incluir() throws ClassNotFoundException, SQLException{
		logDAO = new LogDAO();

		logDAO.insert(getOperacao(), getValor(), getConta(), getAgencia(), getDataOperacao(), getCodigoMovimento(), getCodigoCliente()).execute();
	}

	public ArrayList<Log> consultarestatisticas() throws ClassNotFoundException, SQLException{

		//cria um arrayList para serem armazenado todas as informações do banco de dados
		ArrayList<Log> resultadoPesquisa = new ArrayList<Log>();
		LogDAO logDao = new LogDAO();
	
		ResultSet rs = null;
		rs = logDao.selec().executeQuery();
		
		//insere os valores do banco nos metodos para serem inseridos no arryList
		while(rs.next()){
			//insere os valores do banco nos metodos para serem inseridos no arryList
			Log log = new Log();

			log.setCodigoItemLog(rs.getInt(1));
			log.setCodigoMovimento(rs.getInt(2));
			log.setOperacao(rs.getString(3));
			log.setValor((rs.getInt(4)));
			log.setConta((rs.getInt(5)));
			log.setAgencia((rs.getInt(6)));
			log.setCodigoCliente(rs.getInt(7));
			log.setDataOperacao(rs.getDate(8));
			
			resultadoPesquisa.add(log);
		}
	return resultadoPesquisa;
	}
}

package br.com.usjt.model;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Date;
import java.util.Observable;
import br.com.usjt.DAO.Conexao;
import br.com.usjt.DAO.ContaDAO;
import br.com.usjt.DAO.MovimentoDAO;

public class Movimento extends Observable{
	private int codigoMovimento;
	private Date dataDoMovimento;
	private double valorDaOperacao;
	private Log log;

	public int getCodigoMovimento() {
		return codigoMovimento;
	}

	public void setCodigoMovimento(int codigoMovimento) {
		this.codigoMovimento = codigoMovimento;
		setChanged();
		notifyObservers();
	}


	public Date getDataDoMovimento() {
		return dataDoMovimento;
	}


	public void setDataDoMovimento(Date dataDoMovimento) {
		this.dataDoMovimento = dataDoMovimento;
		setChanged();
		notifyObservers();
	}


	public double getValorDaOperacao() {
		return valorDaOperacao;
	}


	public void setValorDaOperacao(double valorDaOperacao) {
		this.valorDaOperacao = valorDaOperacao;
		setChanged();
		notifyObservers();
	}

	public void geraMovimento(int agencia, int conta, int agenciaDestino, int contaDestino, String tipo) throws ClassNotFoundException, SQLException{
		MovimentoDAO movDAO = new MovimentoDAO();
		ContaDAO contaDAO = new ContaDAO();

		movDAO.insert(conta, getDataDoMovimento(), getValorDaOperacao(),agenciaDestino, contaDestino,tipo).execute();


		int codigoMovimento = movDAO.selecCodigo(conta);
		int codigoCliente = contaDAO.selectCodigoCliente(conta);

		log = new Log();
		log.setAgencia(agencia);
		log.setConta(conta);
		log.setCodigoMovimento(codigoMovimento);
		log.setCodigoCliente(codigoCliente);
		log.setDataOperacao(getDataDoMovimento());
		log.setValor(getValorDaOperacao());
		log.setOperacao(tipo);

		log.incluir();		
	}

	public  ArrayList<Log>  consultarExtratoDias(Date dataInicial, Date dataFinal) throws SQLException{
		//cria um arrayList para serem armazenado todas as informações do banco de dados
		ArrayList<Log> resultadoPesquisa = new ArrayList<Log>();
		Connection conn = new Conexao().connection();

		MovimentoDAO movimentoDAO = new MovimentoDAO();

		ResultSet rs = null;


		try{
			rs = movimentoDAO.selecBetween(dataInicial, dataFinal).executeQuery();
			while (rs.next()){
				Log log = new Log();

				log.setCodigoMovimento(rs.getInt(1));
				log.setConta(rs.getInt(2));
				log.setDataOperacao(rs.getDate(3));
				log.setValor(rs.getDouble(4));
				log.setAgenciaDestino(rs.getInt(5));
				log.setContaDestino(rs.getInt(6));
				log.setCodigoCliente(rs.getInt(7));
				log.setOperacao(rs.getString(9));

				resultadoPesquisa.add(log);
			}
			conn.close();
		}catch (Exception e){
			e.printStackTrace();
			try{
				conn.rollback();
			}catch (SQLException e1){
				System.out.print(e1.getStackTrace());
			}
		}
		return resultadoPesquisa;
	}

	public  ArrayList<Log>  consultarExtratoUltimosDias(int conta) throws SQLException{
		//cria um arrayList para serem armazenado todas as informações do banco de dados
		ArrayList<Log> resultadoPesquisa = new ArrayList<Log>();
		Connection conn = new Conexao().connection();

		MovimentoDAO movimentoDAO = new MovimentoDAO();

		ResultSet rs = null;


		try{
			rs = movimentoDAO.selec(conta).executeQuery();
			while (rs.next()){
				Log log = new Log();

				log.setCodigoMovimento(rs.getInt(1));
				log.setConta(rs.getInt(2));
				log.setDataOperacao(rs.getDate(3));
				log.setValor(rs.getDouble(4));
				log.setAgenciaDestino(rs.getInt(5));
				log.setContaDestino(rs.getInt(6));
				log.setCodigoCliente(rs.getInt(7));
				log.setOperacao(rs.getString(9));

				resultadoPesquisa.add(log);
			}
			conn.close();
		}catch (Exception e){
			e.printStackTrace();
			try{
				conn.rollback();
			}catch (SQLException e1){
				System.out.print(e1.getStackTrace());
			}
		}
		return resultadoPesquisa;
	}
}

package br.com.usjt.model;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Date;

import javax.swing.JOptionPane;
import br.com.usjt.DAO.Conexao;
import br.com.usjt.DAO.ContaDAO;

public class Saque extends Movimento {

	private Dispenser dispenser;
	private int conta, agencia;

	public Saque(Movimento movimento) {
		dispenser = new Dispenser();
	} 

	public int getConta() {
		return conta;
	}

	public int getAgencia() {
		return agencia;
	}
	
	public void setConta(int conta) {
		this.conta = conta;
		setChanged();
		notifyObservers();
	}
	
	public void setAgencia(int agencia) {
		this.agencia = agencia;
		setChanged();
		notifyObservers();
	}

	public void fazerSaque(double valorSacar){
		ContaDAO contaDAO = new ContaDAO();

		double saldoAtual = 0;
		
		try {
			Connection conn = new Conexao().connection();

			ResultSet rs = contaDAO.selec(getConta()).executeQuery();

			while(rs.next()){  
				saldoAtual = Integer.parseInt(rs.getString(5));				 
			}  
			
			double novosaldo = 0;
			if(saldoAtual >= valorSacar){
				dispenser.contarNotas(valorSacar);

				novosaldo = saldoAtual - valorSacar;

				contaDAO.update(getConta(), novosaldo).executeUpdate();	
			  
				Date dataHoje = new Date(); 
				
				setDataDoMovimento(dataHoje);
				
				setValorDaOperacao(valorSacar);
				
				geraMovimento(getAgencia(), getConta(), 0, 0, "Debito em Conta corrente");
				
			}else{
				JOptionPane.showMessageDialog(null, "Saldo insuficiente para saque");
			}
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}

package br.com.usjt.model;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Date;

import javax.swing.JOptionPane;

import br.com.usjt.DAO.ContaDAO;

public class Transferencia extends Movimento{

	private int agenciaDestino, contaDestino, conta;

	public Transferencia(Movimento movimento) {

	}

	public int getAgenciaDestino() {
		return agenciaDestino;
	}

	public void setAgenciaDestino(int agenciaDestino) {
		this.agenciaDestino = agenciaDestino;
		setChanged();
		notifyObservers();
	}

	public int getConta() {
		return conta;
	}

	public void setConta(int conta) {
		this.conta = conta;
	}


	public int getContaDestino() {
		return contaDestino;
	}

	public void setContaDestino(int contaDestino) {
		this.contaDestino = contaDestino;
		setChanged();
		notifyObservers();
	}

	public void fazeTransferencia() throws SQLException, NumberFormatException, IOException, ClassNotFoundException{
		int saldoTitular = 0;
		int saldoContaTransferir = 0;

		FileReader txtConta = new FileReader (""+ getContaDestino());

		BufferedReader entrada = new BufferedReader(txtConta);

		ArrayList<Integer> linhasTxt = new ArrayList<Integer>();
		String texto;

		while((texto = entrada.readLine()) != null){

			linhasTxt.add(Integer.parseInt(texto, 16));//converte de hexadecimal para inteiro

		}

		int conta = linhasTxt.get(0);
		int agencia = linhasTxt.get(1);

		if(agencia == getAgenciaDestino() && conta == getContaDestino()){
			ResultSet rs = null;
			ContaDAO contaDAO = new ContaDAO();
			rs = contaDAO.selectSaldo(getConta()).executeQuery();

			while (rs.next()){
				saldoTitular = rs.getInt(1);
			}

			rs = contaDAO.selectSaldo(getContaDestino()).executeQuery();

			while (rs.next()){
				saldoContaTransferir = rs.getInt(1);
			}

			double novoSaldoContaDestino =  saldoContaTransferir + getValorDaOperacao();
			double novoSaldoContaDebitada = saldoTitular - getValorDaOperacao();

			contaDAO.update(getConta(), novoSaldoContaDebitada).executeUpdate();
			contaDAO.update(getContaDestino(), novoSaldoContaDestino).executeUpdate();

			geraMovimento(agencia, getConta(), getAgenciaDestino(), getContaDestino(), "Transferencia");

			JOptionPane.showMessageDialog(null, "Transferencia Realizada com sucesso");
		}else{
			JOptionPane.showMessageDialog(null, "favor verifique as informacoes da conta a ser creditada");
		}
	}
}

package br.com.usjt.TO;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.io.LineNumberReader;
import java.util.ArrayList;
import java.util.Locale;
import java.util.Observable;
import java.util.ResourceBundle;
import javax.swing.JOptionPane;
import br.com.usjt.view.TelaEntrarComCodigo;
import br.com.usjt.view.TelaGerarCodigo;

public class AcessoTO extends Observable{
	
	
		private int agencia, conta, senha, codigoDeAcesso;
		private FileReader txtConta;
		private boolean validar;
		private ResourceBundle idioma;

		public int getAgencia() {
			return agencia;
		}

		public void setAgencia(int agencia) {
			this.agencia = agencia;
			setChanged();
			notifyObservers();
		}

		public int getConta() {
			return conta;
		}

		public void setConta(int conta) {
			this.conta = conta;
			setChanged();
			notifyObservers();
		}

		public int getSenha() {
			return senha;
		}

		public void setSenha(int senha) {
			this.senha = senha;
			setChanged();
			notifyObservers();
		}

		public int getCodigoDeAcesso() {
			return codigoDeAcesso;		
		}

		public void setCodigoDeAcesso(int codigoDeAcesso) {
			this.codigoDeAcesso = codigoDeAcesso;
			setChanged();
			notifyObservers();
		}

		public ResourceBundle getIdioma() {
			return idioma;
		}

		public void setIdioma(ResourceBundle idioma) {
			this.idioma = idioma;
		}

}


package br.com.usjt.TO;

import java.util.Observable;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Observable;

import javax.swing.JOptionPane;

import br.com.usjt.DAO.Conexao;
import br.com.usjt.DAO.ContaDAO;

public class ContaTO extends Observable{

	private int agencia, saldo, numConta;
	private String nome;



	public String getNome() {
		return nome;
	}
	
	public void setNome(String nome) {
		this.nome = nome;
	}
	
	public int getAgencia() {		
		return agencia;
	}

	public void setAgencia(int agencia) {
		this.agencia = agencia;
		setChanged();
		notifyObservers();
	}

	public int getNumConta() {
		return numConta;
	}

	public void setNumConta(int numConta) {
		this.numConta = numConta;
		setChanged();
		notifyObservers();

	}

	public int getSaldo() {
		return saldo;
	}

	public void setSaldo(int saldo) {
		this.saldo = saldo;
		setChanged();
		notifyObservers();
	}

}


package br.com.usjt.TO;

import java.text.ParseException;
import java.util.Date;
import java.util.Observable;

import br.com.usjt.DAO.DebitoAutomaticoDAO;
import br.com.usjt.model.Movimento;

public class DebitoAutomaticoTO extends Observable{

	private int codigoConsumidor, codDebito;
	private String operadora, tipoDebito;
	private Date dataDebito;
	private double valorDebito;
	private DebitoAutomaticoDAO debDao;
	private Movimento movimento;
	private int numConta, numAgencia;


	public int getCodDebito() {
		return codDebito;
	}

	public void setCodDebito(int codDebito) {
		this.codDebito = codDebito;
	}

	public String getTipoDebito() {
		return tipoDebito;
	}
	public void setTipoDebito(String tipoDebito) {
		this.tipoDebito = tipoDebito;
		setChanged();
		notifyObservers();
	}
	public String getOperadora() {
		return operadora;
	}
	public void setOperadora(String operadora) {
		this.operadora = operadora;
		setChanged();
		notifyObservers();
	}
	public int getCodigoConsumidor() {
		return codigoConsumidor;
	}
	public void setCodigoConsumidor(int codigoConsumidor) {
		this.codigoConsumidor = codigoConsumidor;
		setChanged();
		notifyObservers();
	}
	public Date getDataDebito() {
		return dataDebito;
	}
	public void setDataDebito(Date dataDebito) throws ParseException {    
		this.dataDebito = dataDebito;
	}
	public double getValorDebito() {
		return valorDebito;
	}
	public void setValorDebito(double valorDebito) {
		this.valorDebito = valorDebito;
	}

	public int getNumConta() {
		return numConta;
	}

	private int getAgencia() {
		return numAgencia;
	}

	public void setNumAgencia(int numAgencia) {
		this.numAgencia = numAgencia;
	}

	public void setNumConta(int numConta) {
		this.numConta = numConta;
	}
}

package br.com.usjt.TO;

import java.util.Observable;

public class DispenserTO extends Observable{

	private int nota, quantidade;

	public DispenserTO(int nota, int quantidade) {
		this.nota = nota;
		this.quantidade = quantidade;
	}

	public DispenserTO() {
	}

	public int getNota() {
		return nota;
	}

	public int getQuantidade() {
		return quantidade;
	}

	public void setNota(int nota) {
		this.nota = nota;
		setChanged();
		notifyObservers();

	}

	public void setQuantidade(int quantidade) {
		this.quantidade = quantidade;
		setChanged();
		notifyObservers();
	}
}


package br.com.usjt.TO;

import java.util.Date;
import java.util.Observable;

import br.com.usjt.DAO.LogDAO;

public class LogTO extends Observable{
	private int codigoItemLog, agencia, conta, codigoCliente, codigoMovimento, agenciaDestino, contaDestino;
	private Date dataOperacao;
	private double valor;
	private String operacao;
	private LogDAO logDAO;

	public int getCodigoItemLog() {
		return codigoItemLog;
	}
	public void setCodigoItemLog(int codigoItemLog) {
		this.codigoItemLog = codigoItemLog;
		setChanged();
		notifyObservers();
	}
	
	public int getAgencia() {
		return agencia;
	}
	public void setAgencia(int agencia) {
		this.agencia = agencia;
		setChanged();
		notifyObservers();
	}
	public int getConta() {
		return conta;
	}
	public void setConta(int conta) {
		this.conta = conta;
		setChanged();
		notifyObservers();
	}
	public Date getDataOperacao() {
		return dataOperacao;
	}
	public void setDataOperacao(Date dataOperacao) {
		this.dataOperacao = dataOperacao;
		setChanged();
		notifyObservers();
	}
	public double getValor() {
		return valor;
	}
	public void setValor(double valor) {
		this.valor = valor;
		setChanged();
		notifyObservers();
	}
	
	public int getCodigoCliente() {
		return codigoCliente;
	}
	
	public void setCodigoCliente(int codigoCliente) {
		this.codigoCliente = codigoCliente;
		setChanged();
		notifyObservers();
	}
	
	public int getCodigoMovimento() {
		return codigoMovimento;
	}
	
	public void setCodigoMovimento(int codMovimento) {
		this.codigoMovimento = codMovimento;
		setChanged();
		notifyObservers();
	}
	
	
	public String getOperacao() {
		return operacao;
	}
	
	public void setOperacao(String operacao) {
		this.operacao = operacao;
		setChanged();
		notifyObservers();
	}
	
	public int getContaDestino() {
		return contaDestino;
	}
	
	public void setContaDestino(int contaDestino) {
		this.contaDestino = contaDestino;
		setChanged();
		notifyObservers();
	}
	
	public int getAgenciaDestino() {
		return agenciaDestino;
	}
	
	public void setAgenciaDestino(int agenciaDestino) {
		this.agenciaDestino = agenciaDestino;
		setChanged();
		notifyObservers();
	}
}


package br.com.usjt.TO;

import java.util.Date;
import java.util.Observable;

import br.com.usjt.model.Log;

public class MovimentoTO extends Observable{
	private int codigoMovimento;
	private Date dataDoMovimento;
	private double valorDaOperacao;
	private Log log;

	public int getCodigoMovimento() {
		return codigoMovimento;
	}

	public void setCodigoMovimento(int codigoMovimento) {
		this.codigoMovimento = codigoMovimento;
		setChanged();
		notifyObservers();
	}


	public Date getDataDoMovimento() {
		return dataDoMovimento;
	}


	public void setDataDoMovimento(Date dataDoMovimento) {
		this.dataDoMovimento = dataDoMovimento;
		setChanged();
		notifyObservers();
	}


	public double getValorDaOperacao() {
		return valorDaOperacao;
	}


	public void setValorDaOperacao(double valorDaOperacao) {
		this.valorDaOperacao = valorDaOperacao;
		setChanged();
		notifyObservers();
	}
}


package br.com.usjt.TO;


import br.com.usjt.model.Movimento;

public class SaqueTO extends Movimento {

	private DispenserTO dispenserTO;
	private int conta, agencia;

	public SaqueTO(Movimento movimento) {
		dispenserTO = new DispenserTO();
	} 

	public int getConta() {
		return conta;
	}

	public int getAgencia() {
		return agencia;
	}
	
	public void setConta(int conta) {
		this.conta = conta;
		setChanged();
		notifyObservers();
	}
	
	public void setAgencia(int agencia) {
		this.agencia = agencia;
		setChanged();
		notifyObservers();
	}
}


package br.com.usjt.TO;



public class TransferenciaTO extends MovimentoTO{

	private int agenciaDestino, contaDestino, conta;

	public TransferenciaTO(MovimentoTO movimentoTO) {

	}

	public int getAgenciaDestino() {
		return agenciaDestino;
	}

	public void setAgenciaDestino(int agenciaDestino) {
		this.agenciaDestino = agenciaDestino;
		setChanged();
		notifyObservers();
	}

	public int getConta() {
		return conta;
	}

	public void setConta(int conta) {
		this.conta = conta;
	}


	public int getContaDestino() {
		return contaDestino;
	}

	public void setContaDestino(int contaDestino) {
		this.contaDestino = contaDestino;
		setChanged();
		notifyObservers();
	}

}


package br.com.usjt.DAO;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class Conexao {
	{     
		try{        
			Class.forName("com.mysql.jdbc.Driver");   
		} catch (ClassNotFoundException e){          
			throw new RuntimeException(e);     
		}  
	}  
	public Connection connection() throws SQLException{       
		return DriverManager.getConnection ("jdbc:mysql://localhost/projeto?user=root&password=root" );   
	}
}

package br.com.usjt.DAO;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import br.com.usjt.TO.ContaTO;

public class ContaDAO {
	private PreparedStatement stm;

	public ContaDAO() {
		try {
			stm = null;

		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	public PreparedStatement  selec(int numConta) throws ClassNotFoundException, SQLException {
		Connection conn = new Conexao().connection();	
		try {
			stm = conn.prepareStatement("SELECT * FROM Conta WHERE conta.conta = ?");
			stm.setInt(1, numConta);
		} catch (Exception e) {
			e.printStackTrace();
		}
		
		return stm;
	}
	
	
	public String  innerJoin(int numConta) throws ClassNotFoundException, SQLException {
		Connection conn;
		String nome = "";
		try {
			conn = new Conexao().connection();
			stm = conn.prepareStatement("SELECT nomeCliente from Cliente inner join Conta on Conta.codigoCliente = Cliente.codigoCliente where conta = ?");
			stm.setInt(1, numConta);
			
			ResultSet rs = stm.executeQuery();
			while (rs.next()){
				nome = rs.getString(1);
			}	
			conn.close();
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return nome;
	}
	
	
	public int  selectCodigoCliente(int numConta) throws ClassNotFoundException, SQLException {
		Connection conn;
		int codigoCliente = 0;
		try {
			conn = new Conexao().connection();
			stm = conn.prepareStatement("SELECT codigoCliente FROM Conta WHERE conta.conta = ?");
			stm.setInt(1, numConta);
			
			ResultSet rs = stm.executeQuery();
			while (rs.next()){
				codigoCliente = rs.getInt(1);
			}	
			conn.close();
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return codigoCliente;
	}
	
	public PreparedStatement selectSaldo(int numConta){
		Connection conn;
		try {
			conn = new Conexao().connection();	
			stm = conn.prepareStatement("SELECT saldo FROM Conta WHERE conta.conta = ?");
			stm.setInt(1, numConta);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return stm;
	}

public PreparedStatement update(ContaTO to){
	String sqlUpdate = "UPDATE contaTO  SET numConta=? SET saldo=?";
	try (Connection conn = new Conexao().connection();
	PreparedStatement stm = conn.prepareStatement(sqlUpdate);) {
		stm.setDouble(1, to.getNumConta());
		stm.setInt(2, to.getSaldo());
	} catch (SQLException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
	return stm;
}
}

package br.com.usjt.DAO;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.List;

import br.com.usjt.TO.DispenserTO;
import br.com.usjt.model.Dispenser;


public class DispenserDAO {

	private Connection connection;
	private PreparedStatement stm;

	public DispenserDAO() throws SQLException {
		connection = new  Conexao().connection();
	}

	public boolean resetarDispencher() {
		try {
			PreparedStatement stmt = connection.prepareStatement("UPDATE Dispenser SET QuantidadeDeNotas = 1000 WHERE nota = 10 or Nota = 20");
			stmt.execute();
			stmt = connection.prepareStatement("UPDATE Dispenser SET QuantidadeDeNotas = 500 WHERE Nota = 50");
			stmt.execute();

			connection.close();
			return true;
		} catch (SQLException e) {
			e.printStackTrace();
			return false;
		} finally {
		}
	}

	public PreparedStatement recuperarNotas() {
		Connection conn;
		try {
			conn = new Conexao().connection();
			stm = conn.prepareStatement("SELECT * FROM Dispenser");
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		return stm;
	}

	public boolean Saque(List<DispenserTO> listaNota){
		String sqlSelect = "UPDATE dispenser SET QuantidadeDeNotas = ? WHERE Nota = ?";	
		DispenserTO to = new DispenserTO();
		try {

			for (DispenserTO dispenserTO: listaNota) {

				PreparedStatement stmt = connection.prepareStatement(sqlSelect);{
					stmt.setInt(2, dispenserTO.getNota());
					stmt.setInt(1, dispenserTO.getQuantidade());
					stmt.execute();
				}

				connection.close();

				return true;
			} catch (SQLException e) {
				e.printStackTrace();
				return false;
			}
		}
	}
}
