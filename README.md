import java.util.Scanner;
import java.util.regex.*;
class Juego {
  public static char[][] grid = new char[8][8];                 //Tablero público
  public static Scanner in = new Scanner(System.in);            //Escáner

  public static int winner(){
    /* Esta primera parte es para el modo básico */
    boolean noBlackMoves, noWhiteMoves;
    String firstRow = String.valueOf(grid[0]);
    String lastRow = String.valueOf(grid[7]);
    String mediumRows = new String();
    String board = new String();
    for(int i = 0; i < 8; i++)
      board+= String.valueOf(grid[i]);
    for(int i = 1; i < 7; i++)
      mediumRows += String.valueOf(grid[i]);
    //Casos de tablas: sin movimientos posibles, y sin damas (por eso es el modo básico).
    noBlackMoves = (firstRow.indexOf('n') != -1 && mediumRows.indexOf('n') == -1 && mediumRows.indexOf('b') == -1 && lastRow.indexOf('n') == -1 && board.indexOf('N') == -1);
    noWhiteMoves = (lastRow.indexOf('b') != -1 && mediumRows.indexOf('n') == -1 && mediumRows.indexOf('b') == -1 && firstRow.indexOf('b') == -1 && board.indexOf('B') == -1);
    if(noBlackMoves && noWhiteMoves)
      return 3;
    /* Para el resto de modos */
    boolean noWhitePieces = true, noBlackPieces = true;
    for(int i = 0; i < 8; i++)      //Buscar piezas
      for(char j : grid[i]){
        if(j == 'n' || j == 'N')
          noBlackPieces = false;
        if(j == 'b' || j == 'B')
          noWhitePieces = false;
      }
    if(!noWhitePieces && !noBlackPieces)
      return 0;
    if(noWhitePieces)
      return 1;
    if(noBlackPieces)
      return 2;
    return 0;
  }
  
  public static int[] convertStringtoNum(String s){     //Devuelve array con coordenadas
    s = s.replace("(", "");
    s = s.replace(")", "");
    s = s.replace(",", "");
    int len = s.length();                     //Longitud del array después de quitar los caracteres
    int[] nums = new int[len];
    for(int i = 0; i < len; i++)
      nums[i] = Integer.parseInt(s.substring(i, i + 1));  //Un solo número
    return nums;
  }

  public static boolean validateCoordinates(int[] a){           //Validar coordenadas para comer
    int len = a.length;                                 //Útil
    for(int i = 1; i < len/2; i++){
      int x = a[2*i-2], y = a[2*i-1], nextX = a[2*i], nextY = a[2*i+1];       //Útil
      int xLenB = (grid[a[0]-1][a[1]-1] == 'b' ? nextX - x: (int) Math.abs(nextX - x));   //Por si es o no dama (blancas)
      int xLenN = (grid[a[0]-1][a[1]-1] == 'n' ? x - nextX : (int) Math.abs(nextX - x));   //Por si es o no dama (negras)
      int yLen = (int) Math.abs(nextY - y);   //Valor absoluto para izquierda y derecha
      char between = grid[((x+nextX)/2)-1][((y+nextY)/2)-1];    //El número en medio de a y b es (a + b)/2
      if(grid[a[0]-1][a[1]-1] == 'b' || grid[a[0]-1][a[1]-1] == 'B'){         //Para piezas blancas
        if(xLenB != 2 || yLen != 2){            //Saltos de 2 casillas
          if(grid[a[0]-1][a[1]-1] == 'b' && (xLenB != 1 || yLen != 1))
            return false;
          else if(grid[a[0]-1][a[1]-1] == 'b' && (xLenB == 1 || yLen == 1))
            return true;
          else if(grid[a[0]-1][a[1]-1] == 'B')
            return true;
        }
        if(between != 'n' && between != 'N')
          return false;
      }
      else{                                                    //Para piezas negras
        if(xLenN != 2 || yLen != 2){            //Saltos de 2 casillas
          if(grid[a[0]-1][a[1]-1] == 'n' && (xLenN != -1 || yLen != 1))
            return false;
          else if(grid[a[0]-1][a[1]-1] == 'n' && (xLenN == 1 || yLen == 1))
            return true;
          else if(grid[a[0]-1][a[1]-1] == 'N')
            return true;
        }
        if(between != 'b' && between != 'B')
          return false;
      }
    }
    return true;
  }

  public static void Eat(int[] mov){
    int len = mov.length;
    for(int i = 1; i < len/2; i++){
      int x = mov[2*i-2], y = mov[2*i-1], nextX = mov[2*i], nextY = mov[2*i+1];       //Útil
      grid[((x+nextX)/2)-1][((y+nextY)/2)-1] = '·';    //El número en medio de a y b es (a + b)/2
    }
  }

  public static int[] scanMov(){      //Registrar movimiento introducido por teclado
    int x = 0, y = 0;   //Coordenadas
    //Regexp
    Pattern pat = Pattern.compile("([(][1-8],[1-8][)]){2,12}"); //Patrón regex
    //Input
    String s = new String();
    boolean valid;                  //Para salir del bucle
    do{
      valid = true;
      s = in.nextLine();
      s = s.replace(" ", "");                 //Quitar espacios
      Matcher match = pat.matcher(s);                             //Buscar coincidencia
      boolean found = match.matches();
      if(!found){                             //Ver si coincide
        System.out.println("Input no válido. Formato: ([1-8],[1-8])([1-8],[1-8])");
        valid = false;
      }
      else {                    //Asignar coordenadas y hacer comprobación de base
        int[] pos = convertStringtoNum(s);
        x = pos[0]; y = pos[1];
        if(x > 8 || y > 8 || x < 1 || y < 1){
          System.out.println("Error: fuera del tablero. Volver a introducir:");
          valid = false;
        }
      }
    } while(!valid);
    int[] out = convertStringtoNum(s);        //Hacer lo mismo con el string válido para sacarlo
    return out;
  }

  public static void movWhite(){
    if(winner() != 0)
      return;
    System.out.println("\nBlancas mueven: ");
    int x, y, nX, nY, xLen, yLen;
    boolean legal;                    //Para salir del bucle
    int[] mov;
    do{
      legal = true;
      mov = scanMov();          //Introducir movimiento por teclado
      int len = mov.length;           //Longitud
      x = mov[0]; y = mov[1]; nX = mov[len - 2]; nY = mov[len - 1];     //Más fácil con variables
      yLen = (int) Math.abs(nY - y); xLen = nX - x;     //Helper variables
      if(grid[x - 1][y - 1] != 'b' && grid[x - 1][y - 1] != 'B'){   //Si no hay pieza blanca
        System.out.println("No hay pieza blanca en la casilla seleccionada. Volver a introducir: ");
        legal = false;
      }
      //else if(grid[x - 1][y - 1] == 'b' && (yLen != 1 || xLen != 1)){      //Movimiento ilegal para peones
        //System.out.println("Movimiento ilegal. Volver a introducir:");
        //legal = false;
      //}
      else if(grid[nX - 1][nY - 1] == 'n' || grid[nX - 1][nY - 1] == 'N' ||
       grid[nX - 1][nY - 1] == 'b' || grid[nX - 1][nY - 1] == 'B'){   //Si hay pieza en la casilla final
        System.out.println("Movimiento ilegal: pieza en el camino. Volver a introducir: ");
        legal = false;
      }
      //else if(len == 4)                        //Movimientos normales
        //if(xLen != 1 && yLen != 1){
          //System.out.println("Movimiento ilegal. Volver a introducir");
          //legal = false;
        //}
      else if(xLen != 1 || yLen != 1){                              //Comer
        legal = validateCoordinates(mov);
        if(!legal)
          System.out.println("Movimiento ilegal. Volver a introducir: ");
      }
    } while(!legal);
    grid[x - 1][y - 1] = '·';
    grid[nX - 1][nY - 1] = (grid[x-1][y-1] == 'b' ? 'b' : 'B');
    if(xLen != 1 && yLen != 1)
      Eat(mov);
  }

  public static void movBlack(){
    if(winner() != 0)
      return;
    System.out.println("\nNegras mueven: ");
    int x, y, nX, nY, xLen, yLen;
    boolean legal;                    //Para salir del bucle
    int[] mov;
    do{
      legal = true;
      mov = scanMov();                //Movimiento introducido por teclado
      int len = mov.length;
      x = mov[0]; y = mov[1]; nX = mov[len - 2]; nY = mov[len - 1];     //Más fácil con variables
      yLen = (int) Math.abs(nY - y); xLen = x - nX;     //Helper variables
      if(grid[x - 1][y - 1] != 'n' && grid[x - 1][y - 1] != 'N'){   //Si no hay pieza negra
        System.out.println("No hay pieza negra en la casilla seleccionada. Volver a introducir: ");
        legal = false;
      }
      //else if(grid[x - 1][y - 1] == 'n' && (yLen != 1 || xLen != -1)){           //Movimiento ilegal para peones
        //System.out.println("Movimiento ilegal. Volver a introducir:");
        //legal = false;
      //}
      else if(grid[nX - 1][nY - 1] == 'n' || grid[nX - 1][nY - 1] == 'N' ||
       grid[nX - 1][nY - 1] == 'b' || grid[nX - 1][nY - 1] == 'B'){   //Si hay pieza en la casilla final
        System.out.println("Movimiento ilegal: pieza en el camino. Volver a introducir: ");
        legal = false;
      }
      //else if(len == 4)                        //Movimientos normales
        //if(xLen != 1 && yLen != 1){
          //System.out.println("Movimiento ilegal. Volver a introducir");
          //legal = false;
        //}
      else if(xLen != 1 || yLen != 1){
        legal = validateCoordinates(mov);
        if(!legal)
          System.out.println("Movimiento ilegal. Volver a introducir: ");
      }
    } while(!legal);
    grid[x - 1][y - 1] = '·';
    grid[nX - 1][nY - 1] = (grid[x-1][y-1] == 'n' ? 'n' : 'N');
    if(xLen != 1 && yLen != 1)
      Eat(mov);
  }
  
  public static void genGrid(){
    for(int i = 0; i < 8; i++){
      for(int j = 0; j < 8; j++){
        grid[i][j] = '·';
      }
    }
    //White pieces
    boolean put = true;
    for(int i = 0; i < 3; i++){
      for(int j = 0; j < 8; j++){
        if(put == true)
          grid[i][j] = 'b';
        put = !put;
      }
      put = !put;
    }
    //Black pieces
    put = false;
    for(int i = 7; i > 4; i--){
      for(int j = 0; j < 8; j++){
        if(put == true)
          grid[i][j] = 'n';
        put = !put;
      }
      put = !put;
    }
  }

  public static void printBoard(){
    for(int i = 0; i < 8; i++){
      System.out.printf("%d ", 8 - i);
      for(int j = 0; j < 8; j++){
        System.out.print(grid[8 - i - 1][j] + " ");   //Imprimir de abajo hacia arriba
      }
      System.out.println();
    }
    System.out.print("  ");
    for(int i = 1; i < 9; i++)
      System.out.printf("%d ", i);
  }
  
  public static void basic(){
    genGrid();
    while(winner() == 0){
      printBoard();
      movWhite();
      printBoard();
      movBlack();
    }
    int win = winner();
    if(win == 1)
      System.out.println("\nGanan blancas.");
    else if(win == 2)
      System.out.println("\nGanan negras");
    else if(win == 3)
      System.out.println("\nLa partida es tablas");
  }
  public static void intermediate(){
    genGrid();
    while(winner() == 0){
      printBoard();
      movWhite();
      String lastRow = String.valueOf(grid[7]);
      if(lastRow.indexOf('b') != -1)              //Coronar pieza blanca
        grid[7][lastRow.indexOf('b')] = 'B';
      printBoard();
      movBlack();
      String firstRow = String.valueOf(grid[0]);
      if(firstRow.indexOf('n') != -1)             //Coronar pieza negra
        grid[0][firstRow.indexOf('n')] = 'N';
    }
    int win = winner();
    if(win == 1)
      System.out.println("\nGanan blancas.");
    else if(win == 2)
      System.out.println("\nGanan negras");
    else if(win == 3)
      System.out.println("\nLa partida es tablas");
  }
  public static void main(String[] args) {
    System.out.println("¡Bienvenido! Elija el modo en el que quiere jugar:");
    System.out.println("Básico(1)\tIntermedio(2)\tAvanzado(3)");
    boolean stop;                   //Puede usarse solo una variable
    do{
      do{
        stop = true;
        String choice = in.nextLine();
        switch (choice){
          case "1":
            basic();
            break;
          case "2":
            intermediate();
            break;
          case "3":
            break;
          default:
            System.out.println("Lo siento, vuelva a introducir el modo: ");
            stop = false;
        }
      }while(!stop);
      boolean choice;
      do{
        choice = true;
        System.out.println("¡Gracias por jugar!¿Desea comenzar otra partida? (S/n)");
        String choice1 = in.nextLine();
        choice1 = choice1.substring(0,1);   //Para reducir casos en el switch
        switch (choice1){
          case "":
          case "S":
          case"s":
            stop = false;
            choice = true;
            break;
          case "n":
          case "N":
            stop = true;
            choice = true;
            break;
          default:
            System.out.println("Lo siento, vuelva a introducir su elección: ");
            choice = false;
        }
      }while(!choice);
    }while(!stop);
  }
}
