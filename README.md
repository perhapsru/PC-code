# PC-code。
package 递归分析;


import java.awt.*;
import java.io.*;
import java.util.ArrayList;
import java.awt.event.*;

@SuppressWarnings("serial")
public class Analysis extends Frame implements ActionListener{
     private Frame f;
     private TextArea ta;   
     private TextArea tb;
     private TextArea tc;
     private Button btn;
     private Button btn1;
     private Button btn2;
     private Button btn3;
     private FileDialog fd;
     private File file1 = null;
     ArrayList<String> list = new ArrayList<String>();

     public Analysis()
      {
    	 
    	   btn = new Button("打开");
           btn2 = new Button("清空");
           btn1 = new Button("开始分析");
           btn3 = new Button("递归下降分析");
           ta = new TextArea(40,40);
           tb = new TextArea(40,40);
           tc = new TextArea(40,40);

           btn.addActionListener(this);
           btn2.addActionListener(this);
           btn1.addActionListener(this);
           btn3.addActionListener(this);
       }
        
     public void actionPerformed(ActionEvent e){
          if (e.getSource() == btn) {  
                fd = new FileDialog(f,"Open",FileDialog.LOAD);
                fd.setVisible(true);                             
                try {
               
                    file1 = new File(fd.getDirectory(),fd.getFile());
                    FileReader fr = new FileReader(file1);
                    BufferedReader br = new BufferedReader(fr);
                    String aline = "";
                    while ((aline=br.readLine()) != null)
                    {
                    	if (aline.matches("^\\s*$")) continue;//   ^是开始      \s是空白     *表示0个或多个     |是或者      $是结尾    g表示全局                       
                    	ta.append(aline+'\n');
                    }
                    
                    BufferedWriter bw = null;
                    try {
                        OutputStream os = new FileOutputStream("D:\\测试文件.txt");
                        bw = new BufferedWriter(new OutputStreamWriter(os));
                        for (String value : ta.getText().split("\n")) {
                            bw.write(value);
                            bw.newLine();//换行
                        }
                    } catch (IOException e1) {
                        e1.printStackTrace();
                    } finally {
                        if (bw != null) {
                            try {
                                bw.close();
                            } catch (IOException e1) {
                                e1.printStackTrace();
                            }
                        }
                    }                   
                    fr.close();
                    br.close();
                  }
               catch (IOException ioe){
               
                 ioe.printStackTrace();
                } 
              }
         if (e.getSource() == btn2){
        	  ta.setText("");
        	  tb.setText("");
        	  tc.setText("");
        	  File file  = new File("D:\\测试文件.txt");
        	  if(file.exists())
        		  file.delete();
          }
         if (e.getSource() == btn1){
        	 try {
				Scanner scanner = new Scanner("D:\\测试文件.txt");
				scanner.scanning();
			} catch (Exception e1) {
				// TODO Auto-generated catch block
				e1.printStackTrace();
			}   
        	
         }   
         if(e.getSource() == btn3){
        	 Recursion r = new Recursion(list);
        	 r.start();
        //	 for(int i=0;i<list.size();i++)
        //	 tc.append(list.get(i)+"\n");
        	 
         }
      }
                     
     public static void main(String []args){
           Analysis iof = new Analysis();
           iof.show();
      }   
     public void show()
         {
        	Toolkit kit = Toolkit.getDefaultToolkit();
            Dimension screenSize = kit.getScreenSize();
        	int screenWidth = screenSize.width;
        	int screenHeight = screenSize.height;
            f = new Frame();       
            f.setTitle("词法分析器");
            f.setSize(screenWidth,screenHeight);  
            f.setLayout(new FlowLayout(FlowLayout.LEFT,20,50));
            f.add(btn);  
            f.add(ta); 
            f.add(btn1);
            f.add(tb);
            f.add(btn3);
            f.add(tc);
            f.add(btn2);
            f.addWindowListener(new WindowAdapter(){
	               public void windowClosing(WindowEvent evt){ 
	               f.setVisible(false);         
	               f.dispose();            
	               System.exit(0);           
	            }
                });
           f.setVisible(true);                
    }
     
     class Scanner {	
    		//定义DFA中的所有状态表
    		private static final int Start = 1;
    		private static final int Num = 2;
    		private static final int WORD = 3; 
    		private static final int Done = 4;
    		
    		//保留字
    		private String [] keyWords = new String[]{
    				"if","then","else","end","repeat","until","read","write"
    		};
    		
    		//特殊字符
    		private String [] special = {
    				"+","-","/","*","=","<","{","}",";","(",")"
    		};   		
    		//源代码文件输入流
    		private BufferedReader sourceFile;
    		//扫描行的最大字符数
    		private static final int BUF_SIZE = 1024;
    	    //当前行的字符长度
    		private int bufSize = 0;
    		//当前行
    		private String thisLine;
    		//当前扫描行的字符序列
    		private char[] lineBuf = new char[BUF_SIZE];
    		//当前扫描的行数
    		private int lineNum = 0;
    		//当前行的字符下标
    		private   int charPos = 0;
    		//是否已达文件尾
    		private boolean isEndOfFile = false;  
    		
    		boolean end = false;

    		//初始化 并读取源代码文件
    		public Scanner (String filePath) throws Exception{
    			this.sourceFile = new BufferedReader(new FileReader(filePath));
    		}		
    		//开始扫描直到读取到文件的结束符
    		public void scanning()throws Exception{
    			while(!isEndOfFile){
    				getToken();
    				
    			}
    			tb.append("分析结束");
    		}   		
    		//获取下一个字符
    		private char getNextChar() throws Exception{
    			char nextChar = '\0'; 	
    			
    			if(!(charPos  < bufSize)){  
    			
    				if((thisLine = sourceFile.readLine()) != null){
    					lineNum++;//扫描的行数+1
    					tb.append("————————  "+"第"+lineNum +"行"+":" +thisLine+"  ————————"+'\n');
    			
    					thisLine += " ";
    					lineBuf = thisLine.toCharArray(); 
    					bufSize = thisLine.length();
    					charPos = 0;
    					nextChar = lineBuf[charPos++];//
    				}else{
    					isEndOfFile = true;
    					nextChar = '\0';
    				}
    			}else{
    				nextChar = lineBuf[charPos++];
    			}
    			
    			return nextChar;
    		}
    		
    		//按step取消获取下一个字符，退位
    		private void unGetNextChar(int step){
    			if(!isEndOfFile){
    				charPos -= step;
    			}
    		}
    			
    	    //获取token
    		private String getToken() throws Exception{
    			// TODO Auto-generated method stub
    			String tokenStr = "";
    			String currentToken = "";//初始化当前token
    			
    			int currentState = Start;//定义初态,状态参量，通过判断定义为最开始的状态集中的一个
    			boolean isSave;
    			
    			
    			//不能同时为终态和结束
    		while(currentState != Done && (!isEndOfFile)){
    				char c = getNextChar();
    				isSave = true;//判断符号，需要继续判断以助于接着往后继续判断    				
    				end = false;
    				switch(currentState){//通过case给currentState赋值，改变状态 				
    				case Start:
    					if(isDigit(c)){
    						currentState = Num;
    					}else if(isLetter(c)){
    						currentState = WORD;
    					}else if(c ==' ' ||c == '\t' ||c =='\n'){//若是' '或'\t'或 \n'，直接跳过,继续判断  		
    						isSave = false;
    						end = true;
    					}else if(c == '\0'){
    						end = true;
    						currentState = Done;
    					}else if(c == '+'||c =='-'||c =='*'||c =='/'||c =='='||c =='<'||c =='{'||c =='}'||c ==';'||c == '('|| c == ')'){
    						currentState = Done;
    					}else if(c == '\0'){
    						currentState =Done;
    					}else {
    						currentState = Done;
    					
    	                   	  }
    					break;			
    				case WORD:
    					if(!isLetter(c)){    					
    						currentState = Done;
    						unGetNextChar(1);
    						isSave = false;    						
    					} 
    					if(isDigit(c)){   						
    						currentState = Done;
    						isSave = false; 
    						tb.append(" "+"第"+lineNum+"出现错误：数字字母混合出现"+'\n'+" 其中："+'\n');  						
    					}
    					break;
    				
    				case Num:
    					if(!isDigit(c)){
    						currentState = Done;
    						unGetNextChar(1);//回退一个字符
    						isSave = false;//继续判断   					
    					}
    					if(isLetter(c)){     					
    						currentState = Done;
    						isSave = false; 						
    						tb.append(" "+"第"+lineNum+"出现错误：数字字母混合出现"+'\n'+" 其中："+'\n');  						
    					}
    					break;	
    				default:
    					currentState = Done;
    					break;
    				}
    				if(isSave){
    					tokenStr += c;
    				}
    				if(currentState == Done){
    					currentToken = tokenStr;
    					if(!end)
    					list.add(currentToken);
    					printToken(currentToken);
    				}
    		
    			}
    		   			
    		return currentToken;    			
    		}

    	
    		private boolean isDigit(char c){
    				if('0'<= c&&c<='9'){
    					return true;
    				}
    				return false;
    			}
    	
    		private boolean isLetter(char c){
    				if(('a'<= c && c <='z') ||('A'<=c && c<='Z')){
    					return true;
    				}
    				return false;
    			} 
    
    		private boolean isWord(String token){
				    boolean flag = true;
				    char [] chs = token.toCharArray();
				    int len = chs.length;
				    for(int i=0;i<len;i++){
				    	if(!isLetter(chs[i])){
				    		flag = false;
				    	}
				    }
				    return flag;
			} 
    	
		    	
   
    		private boolean isNum(String token){
    				boolean flag = true;
    				char [] chs = token.toCharArray();
    				int len = chs.length;
    				for(int i = 0;i< len;i++){
    					if(!isDigit(chs[i])){
    						flag = false;
    					}
    				}
    				return flag;
    			}
   
    		private boolean isSpecial(String token){
    				int len = special.length;
    				boolean flag = false;
    				for(int i = 0;i<len;i++){
    					if(token.equals(special[i])){
    						flag = true;
    					}
    				}
    				return flag;
    			}
    	
    	    private boolean isKeyWord(String token){
    				boolean flag = false;
    				int len = keyWords.length;
    				for(int i = 0;i<len;i++){
    					if(keyWords[i].equals(token)){
    						flag = true;
    					}			
    				}
    				return flag;
    			}
    	
    		
    		private void printToken(String Token) {
    			// TODO Auto-generated method stub

    			String str = "";
    			if(isKeyWord(Token) )
    			{
    				str = " "+Token+"为保留字";
    			}else if(isSpecial(Token))
    			{
    				str =" "+Token+"为特殊符号";
    			}else if(isNum(Token))
    			{
    				str =" "+Token+"为数";	
    			}else if(isWord(Token)){
    				str =" "+Token+"为标识符";
    			}
    	//		else {
    	//			str =" 第"+lineNum+"行出现错误:错误符号";
    	//		}
    			tb.append(str+'\n');
    			tb.append("\n");
    			
    			
    		}
    	}
     
     class Recursion {
    		public String []grammar;
    		public int startPos=0;
    		public String result[];
    		public int pos=0;
    	
    		   		
    		public void start()
    		{		
    			result[pos++] = "program";
    			program(grammar[0]);
    			show();

    		}	
    		public Recursion(ArrayList<String> list)
    		{
    			result=new String[100];
    			grammar=new String[list.size()];
    			for(int i=0;i<list.size();i++)
    			{
    				//if(list.get(i)!=null)
    				grammar[i]=list.get(i);
    			}		
    		}
    			
    		public  String program(String str)
    		{
    			result[pos++]="program ";
    			result[pos-1]=result[pos-1].replaceFirst("program", "block");
    			str=read(startPos);
    			
    			if(!(block(str)).equals("error"))
    			{
    				
    				return "s";
    			}
    			return"error";
    		}
    		
    		
    		public void show()
    		{
    			String str="";
    			for(int i=1;i<pos;i++)
    			{
    				str+=result[i]+"\n";
    				tc.append("->"+result[i]+"\n");
    			}

    		}

    		
    		public  String block(String str)
    		{	
    			str=read(startPos);
    			result[pos++]=result[pos-2];
    			result[pos-1]=result[pos-1].replaceFirst("block", "{stmts}");

    			if(read(startPos).equals("{"))
    			{  
    				startPos++;
    				if(!(stmts(read(startPos))).equals("error"))
    				{  				
    					startPos++;
    					if(read(startPos).equals("}"))
    					{
    						return "{stmts}";
    					}
    					return "error";
    				}
    				return "error";
    			}		
    			return "error";
    		}
    	private String stmts(String str) {			
    			if(!read(startPos).equals("}"))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("stmts", "stmt stmts");			
    				if(!(stmt(read(startPos))).equals("error"))
    				{
    					
    					startPos++;
    					if(!(stmts(read(startPos))).equals("error"))
    					{													
    						return "stmt stmts";
    					}
    					return "error";			
    				}
    				
    				return "null";
    			}
    			result[pos++]=result[pos-2];
    			result[pos-1]=result[pos-1].replaceFirst("stmts", "");
    			startPos--;	
    			return "null";
    		}
    		private String stmt(String str) {

    			str=read(startPos);
    			
    			if((read(startPos).equals("i"))||str.equals("sum"))
    			{	
    				
    				startPos++;
    				
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("stmt", "id=expr;");
    				if(read(startPos).equals("="))
    				{

    					startPos++;
    					if((expr(read(startPos)))!="error")
    					{				
    						startPos++;
    						if(read(startPos).equals(";"))
    						{
    							return "id=expr;";
    						}
    						return "error";
    					}
    					return "error";
    				}
    				return "error";
    			}
    			
    			else if(read(startPos).equals("if"))
    			{
    				
    				startPos++;
    				
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("stmt", "if(bool)stmt stmt'");
    				if(read(startPos).equals("("))
    				{
    					startPos++;
    					if(!((bool(read(startPos)))).equals("error"))
    					{

    						startPos++;
    						if(read(startPos).equals(")"))
    						{
    							startPos++;
    							if(!(stmt(read(startPos))).equals("error"))
    								{

    									startPos++;
    									if(!(stmt1(read(startPos))).equals("error"))
    									{
    										return "if(bool)stmt else stmt";
    									}
    									return "error";
    								}
    							return"error";
    						}
    						return "error";
    					}
    					return "error";
    				}
    				return "error";
    			}
    			else if(read(startPos).equals("while"))
    			{
    				startPos++;
    				
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("stmt", "while(bool)stmt");
    				if(read(startPos).equals("("))
    				{

    					startPos++;
    					if(!(bool(read(startPos))).equals("error"))
    					{
    						startPos++;
    						if(read(startPos).equals(")"))
    						{
    							startPos++;
    							if(!(stmt(read(startPos))).equals("error"))
    							{
    								return "s";
    							}
    							return "error";
    						}
    						return "error";
    					}
    					return "error";
    				}
    				return "error";
    			}
    			else if(read(startPos).equals("do"))
    			{

    				startPos++;
    				
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("stmt", "do stmt while(bool)");
    				if(!(stmt(read(startPos))).equals("error"))
    				{
    					startPos++;
    					if(read(startPos).equals("while"))
    					{ 

    						startPos++;
    						if(read(startPos).equals("("))
    						{
    							startPos++;
    							if(!(bool(read(startPos))).equals("error"))
    							{
    								startPos++;
    								if(read(startPos).equals(")"))
    								{

    									return "do stmt while(bool)";
    								}
    								return "error";
    							}
    							return "error";
    						}
    						return "error";
    					}
    					return "error";
    				}
    				return "error";
    			}
    			else if(read(startPos).equals("break"))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("stmt", "break");
    				
    				return "s";
    			}
    			else if(read(startPos).equals("{"))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("stmt", "block");
    				if(!(block(str)).equals("error"))
    				{

    					return "s";
    				}
    				return "error";
    			}
    			return "error";		
    		}
    		
    		private String stmt1(String str) {
    			str=read(startPos);
    			if(str.equals("else"))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("stmt'", "else stmt");			

    				startPos++;
    				if(!(stmt(read(startPos))).equals("error"))
    				{							
    			
    					return "s";
    				}
    				return "error";
    			}
    			result[pos++]=result[pos-2];
    			result[pos-1]=result[pos-1].replaceFirst("stmt'", "");
    			startPos--;
    			return "null";
    		}
    		private String bool(String str)
    		{
    			String s;
    			str=read(startPos);
    			result[pos++]=result[pos-2];
    			result[pos-1]=result[pos-1].replaceFirst("bool", "expr bool'");
    			if(!(s=expr(str)).equals("error"))
    			{
    				
    				
    				startPos++;
    				if(!(s=bool_1(read(startPos))).equals("error"))
    				{
    					
    					return "s";
    				}
    				return "error";
    			}
    			return "error";
    		}
    		private String bool_1(String str) {
    			str=read(startPos);
    			
    			if(str.equals("<"))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("bool'", "<expr");
    			
    				startPos++;
    				if(!(expr(read(startPos))).equals("error"))
    				{
    					return "s";
    				}
    				return"error";
    			}
    			else if(str.equals("<="))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("bool'", "<=expr");
    				
    				startPos++;
    				if(!(expr(read(startPos))).equals("error"))
    				{
    					
    					return "s";
    				}
    				return"error";
    			}
    			else if(str.equals(">"))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("bool'", ">expr");
    				
    				startPos++;
    				if(!(expr(read(startPos))).equals("error"))
    				{
    					
    					return "s";
    				}
    				return"error";
    			}
    			else if(str.equals(">="))
    			{
    				
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("bool'", ">=expr");
    				
    				startPos++;
    				if(!(expr(read(startPos))).equals("error"))
    				{
    				
    					return "s";
    				}
    				return"error";
    			}
    			result[pos++]=result[pos-2];
    			result[pos-1]=result[pos-1].replaceFirst("bool'", "");
    			startPos--;
    			return "null";
    		}

    		private String expr(String str)
    		{
    			result[pos++]=result[pos-2];
    			result[pos-1]=result[pos-1].replaceFirst("expr", " term expr'");
    			str=read(startPos);
    			if(!(term(str)).equals("error"))
    			{
    				
    				startPos++;
    				if((expr1(read(startPos)))!="error")
    				{
    					
    					return "s";
    				}
    				return "error";
    			}
    			return "error";
    		}
    			
    		
    		private String expr1(String str) {
    			str=read(startPos);
    			
    			
    			if(str.equals("+"))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("expr'", "+term");
    				
    				startPos++;
    				if(!(term(read(startPos))).equals("error"))
    				{
    				
    					return "s";
    				}
    				return "error";
    			}
    			else if(str.equals("-"))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("expr'", "-term");
    				
    				startPos++;
    				if(!(term(read(startPos))).equals("error"))
    				{
    				
    					return "s";
    				}
    				return "error";
    			}
    			result[pos++]=result[pos-2];
    			result[pos-1]=result[pos-1].replaceFirst("expr'", "");
    			startPos--;
    			return "null";
    		}
    		public String term(String str)
    		{
    			result[pos++]=result[pos-2];
    			result[pos-1]=result[pos-1].replaceFirst("term", "factor term'");
    			str=read(startPos);
    			if(!(factor(str)).equals("error"))
    			{
    				
    				startPos++;
    				if(!(term1(read(startPos))).equals("error"))
    				{
    					
    					return "s";
    				}
    				return "error";
    			}
    			return"error";
    		}
    		private String term1(String str) {
    			str=read(startPos);
    			
    			if(str.equals("*"))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("term'", "*factor");
    				
    				startPos++;
    				if(!(factor(read(startPos))).equals("error"))
    				{
    					
    					
    					return "s";
    				}
    				return "error";
    			}
    			if(str.equals("/"))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("term'", "/factor");
    				
    				startPos++;
    				if(!(factor(read(startPos))).equals("error"))
    				{
    									
    					return "s";				
    				}
    				return "error";
    			}
    			result[pos++]=result[pos-2];
    			result[pos-1]=result[pos-1].replaceFirst("term'", "");
    			startPos--;
    			return "null";
    		}
    		public String factor(String str)
    		{
    			
    			str=read(startPos);
    			
    			if(str.equals("("))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("factor", "(expr)");		
    				startPos++;
    				
    				if(!(expr(read(startPos))).equals("error"))
    				{								
    					startPos++;
    					if(read(startPos).equals(")"))
    					{						
    						return "s";
    					}
    					return "error";
    				}
    				return "error";
    			}
    			else if(isId(str))
    			{
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("factor", "id");		
    			
    				return "s";
    			}
    			else if(isNum(str))
    			{
    				
    				result[pos++]=result[pos-2];
    				result[pos-1]=result[pos-1].replaceFirst("factor", "num");
    				return "s";
    			}
    			return "error";
    		}
    		public boolean isDigit(char c)
    		{
    			if(c>='0'&&c<='9')
    			{
    				return true;
    			}
    			return false;
    		}
    		public boolean isNum(String num)
    		{
    			char chr[]=num.toCharArray();
    			for(int i=0;i<chr.length;i++)
    			{
    				if(!isDigit(chr[i]))return false;
    			}
    			return true;
    		}
    		public boolean isLetter(char letter)
    		{
    			if(letter>='a'&&letter<='z')return true;
    			return false;
    		}
    		public boolean isId(String letter)
    		{
    			char []c=letter.toCharArray();
    			for(int i=0;i<c.length;i++)
    			{
    				if(!isLetter(c[i]))return false;
    			}
    			return true;
    		}
    		public String getNum(String num)
    		{		
    			if(isNum(num))return num;
    			return null;
    		}
    		public String getVariable(String variable)
    		{
    			if(isId(variable))return variable;
    			return null;
    		}
    		
    		public String read(int position)
    		{		
    			return grammar[position];
    		}

 /**   		public static void main(String args[])
  *  		{
  *  			String s[] = {"{","i","=","5",";","while","(","i","<","100",")","{","sum","=","sum","+","2",";","i","=","i","+","2",";","}","}"};
  *   			Recursion r = new Recursion(s);
  *  	        r.start();
  *   		}
  *   	}
**/

}
}
