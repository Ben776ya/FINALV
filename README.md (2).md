
# Main Window Implementation :
##  Qt Main Window Framework
_**Content Taken from**  [Here](https://doc.qt.io/qt-5/qmainwindow.html#details)_

A main window provides a framework for building an application's user interface. Qt has QMainWindow and its [related classes](https://doc.qt.io/qt-5/widget-classes.html#main-window-and-related-classes) for main window management. QMainWindow has its own layout to which you can add [QToolBar](https://doc.qt.io/qt-5/qtoolbar.html)s, [QDockWidget](https://doc.qt.io/qt-5/qdockwidget.html)s, a [QMenuBar](https://doc.qt.io/qt-5/qmenubar.html), and a [QStatusBar](https://doc.qt.io/qt-5/qstatusbar.html). The layout has a center area that can be occupied by any kind of widget. You can see an image of the layout below.


![mainwindowlayout](https://user-images.githubusercontent.com/85891554/146638032-65831a11-f57a-4125-8a42-36038506448e.png)  

  **Note:** Creating a main window without a central widget is not supported. You must have a central widget even if it is just a placeholder.
  

 
  # SpreadSheet
  ## Context
  
  A spreadsheet is a computer application for organization, analysis, and storage of data in tabular form. Spreadsheets were developed as computerized analogs of paper accounting worksheets.The program operates on data entered in cells of a table. Each cell may contain either numeric or text data.
  
 We will try to make a spreadsheet ourselves and enhance it with some valuable tools .
 
  ![enter image description here](https://cdn.discordapp.com/attachments/403688901111447552/924707221957398528/1.PNG)
  

 ## StatusBar
 The updateStatusBar takes two parameters **Rows & Columns**
 ```cpp
 void updateStatusBar(int, int) 
```
Here is the implementation of this function:
 ```cpp
void SpreadSheet::updateStatusBar(int row, int col)
{
    QString cell{"(%0, %1)"};
   cellLocation->setText(cell.arg(row+1).arg(col+1));

}
```
>Show the current selected cell in the status bar

Connecting each button to its purpose of creation 
 ```cpp
void SpreadSheet::makeConnexions()
{

// --------- Connexion for the  select all action ----/
connect(all, &QAction::triggered,
        spreadsheet, &QTableWidget::selectAll);

// Connection for the  show grid
connect(showGrid, &QAction::triggered,
        spreadsheet, &QTableWidget::setShowGrid);

//Connection for the exit button
connect(exit, &QAction::triggered, this, &SpreadSheet::close);

//connectting the chane of any element in the spreadsheet with the update status bar
connect(spreadsheet, &QTableWidget::cellClicked, this,  &SpreadSheet::updateStatusBar);
}
```
## Go Cell
Creating the UI On the designer


![enter image description here](https://cdn.discordapp.com/attachments/403688901111447552/924707221454061588/2.PNG)

Now time to create The Slots and link them with their associated buttons through the connect command .
```cpp
 private slots:
 void goCellSlot();            // Go to a Cell slot
```

```cpp
connect(goCell, &QAction::triggered, this, &SpreadSheet::goCellSlot);
```
Now We will implement the goCellSlot() function:
```cpp
void SpreadSheet::goCellSlot(){
    // creer un dialog
    GoDialog d;
     //executer le dialog
    auto reply=d.exec();

    //verifier si le dialog a ete acceplte
    if(reply == GoDialog::Accepted)
    {


        // extraire le text
        QString cell = d.getCell();
        //extraire la ligne

        int row =  cell[0].toLatin1()- 'A';

        //Extraire la colonne

        cell = cell.remove(0,1);
        int col = cell.toInt()-1;

        // changer la cellule
        statusBar()->showMessage("Changing the current cell" , 2000);
        spreadsheet->setCurrentCell(row, col);


     }

}
```




## Find Dialog
We seek to create a search engine ,which will verify whether or not a specific content exists in our spreadsheet . 
First we Create its UI as usual for the user


![ ](https://cdn.discordapp.com/attachments/403688901111447552/924707221651210260/3.PNG)

Now time to create The Slots and link them with their associated buttons through the connect command .
```cpp
 private slots:
 void findSlot();            // Go to a Cell slot
```

```cpp
connect(find, &QAction::triggered, this, &SpreadSheet::findSlot);
```
Now We will implement the goCellSlot() function wich search for a cell that contains the desired content :
```cpp
void SpreadSheet::findSlot(){
    FindDialog dialog;
     auto reply= dialog.exec();
     auto found= false;

     if(reply == FindDialog::Accepted)
     {
         QString text= dialog.getText();
         for(auto i=0; i<spreadsheet->rowCount(); i++)
         {
             for(auto j=0; j<spreadsheet->columnCount(); j++)
             {
                 if(spreadsheet->item(i,j))
                 {
                   QString searchword = spreadsheet->item(i,j)->text();
                    if(searchword.contains(text))
                    {
                        spreadsheet->setCurrentCell(i,j);
                        found= true;



                    }


                }
}
         }
         if(!found)
               QMessageBox::information(this, "Error", "No such word or expression");


     }
     }
```
_Fail CASE_

![Capture d’écran 2021-12-18 200628](https://user-images.githubusercontent.com/85891554/146653025-8af0b808-5d4e-492e-a08e-8c2a4ee0828f.png)


## Saving files
Saving the content of our spreadsheet in a text file **.txt**
- We will add the declaration in the header file
```cpp
 protected:
     void saveContent(QString filename);
```
- For the implementation, we will using two classes:
  - [QFile](https://doc.qt.io/qt-5/qfile.html) Editing files.
  - [QTextStream](https://doc.qt.io/qt-5/qtextstream.html) Manipulating the content .
-Here is the complete implementation of this function:
```cpp
void SpreadSheet::saveContent(QString filename) {

    QFile file(filename);

    //2 ouvrir le fichier en mode write
    if(file.open(QIODevice::WriteOnly))
    {
        QTextStream out(&file);

         //parcourir les cellules et sauvegarder leur contenu
        for (int i=0;i<spreadsheet->rowCount() ;i++ ) {
            for(int j=0; j<spreadsheet->columnCount();j++){
                auto cell = spreadsheet->item(i,j);
                if(cell)
                    out << i<< "," << j << "," << cell->text() << endl;
            }

        }
    }


    // fermer le fichier
    file.close();
}
```
## Save File action
Now time to create The Slots and link them with their associated buttons through the connect command .
```cpp
private slots:
    void saveSlot(); 
```

```cpp
 connect(save, &QAction::triggered, this, &SpreadSheet::saveSlot);
```
the implementation of the slot
```cpp
void SpreadSheet::saveSlot()
{
    //Creating a file dialog to choose a file graphically
    auto dialog = new QFileDialog(this);

    //Check if the current file has a name or not
    if(currentFile == "")
    {
       currentFile = dialog->getSaveFileName(this,"choose your file");

       //Update the window title with the file name
       setWindowTitle(currentFile);
    }

   //If we have a name simply save the content
   if( currentFile != "")
   {
           saveContent(currentFile);
   }
}
```
we create another slot to save it as **.csv**
```cpp
void SpreadSheet::saveascsvslot() {
// tester si on possede un nom de fichier
if(!currentFile)
{
   
    QFileDialog d;

    // get the filename
    auto filename = d.getSaveFileName();
    // change the current file name
    currentFile = new QString(filename);
    // mettre a jour le titre de le fenetre
    setWindowTitle(*currentFile);

}
// call the savecsvcontent function
saveascsvContent(*currentFile);

}
```
the implementation of the savecsvContent function

```cpp
void SpreadSheet::saveascsvContent(QString filename) {
    QFile file(filename);

   int rows = spreadsheet->rowCount();
   int columns = spreadsheet->columnCount();
   if(file.open(QIODevice::WriteOnly )) {

       QTextStream out(&file);


   for (int i = 0; i < rows; i++) {
       for (int j = 0; j < columns; j++) {

           auto cell = spreadsheet->item(i,j);
           if(cell)
               out<< cell->text()<<",";
           else
               out<< " "<<",";

       }
        out<< Qt::endl;

   }


   file.close();
}
}

```

# Text Editor

A **text editor** is a type of [computer program](https://en.wikipedia.org/wiki/Computer_program "Computer program") that edits [plain text](https://en.wikipedia.org/wiki/Plain_text "Plain text"). Such programs are sometimes known as "**notepad**" software.
## Design the text editor

We will use the Designer to make an interface ( the easy way ) .


![enter image description here](https://cdn.discordapp.com/attachments/403688901111447552/924717403798118400/4.PNG)

## Adding actions

Adding the actions to the buttons available in the menubar.

```cpp
void textEditor::on_action_Copy_triggered()
{
    ui->plainTextEdit->copy();
    // update the status bar
    ui->statusbar->showMessage("Copying the current selection");

}
```

>_Copy_

```cpp
void textEditor::on_action_Paste_triggered()
{
   ui->plainTextEdit->paste();
   ui->statusbar->showMessage("Pasting the previous copied selection");

}
```

>_Paste_

```cpp
void textEditor::on_action_Cut_triggered()
{
    ui->plainTextEdit->cut();
    ui->statusbar->showMessage("Cutting the current selection");

}
```

>_Cut_

  

```cpp
void textEditor::on_actionOpen_triggered()
{
    QString filename= QFileDialog::getOpenFileName(this,"Open the file");
    QFile file(filename);
    currentFile = filename;
    if(!file.open(QIODevice::ReadOnly | QFile::Text))
        QMessageBox::warning(this,"Warning","Cannot open file :"+file.errorString());
    setWindowTitle(filename);
    QTextStream in(&file);
    QString text = in.readAll();
    ui->plainTextEdit->setPlainText(text);
    file.close();
    ui->statusbar->showMessage("Opening the chosen file...");
}
```
> Open




```cpp
void textEditor::on_actionNew_triggered()
{
    currentFile.clear();
    currentFile=nullptr;
    ui->plainTextEdit->setPlainText(QString());
    ui->statusbar->showMessage("Creating a new file");
}
```
> New



Adding the Save Function , same as the previous project

```cpp
void textEditor::on_actionSave_triggered()
{
    auto dialog = new QFileDialog(this);
    if(currentFile == "")
    {
      currentFile = dialog->getSaveFileName(this,"choose your file");
       setWindowTitle(currentFile);
    }
   if( currentFile != "")
   {
           saveContent(currentFile);
   }
}
```



```cpp
void textEditor::saveContent(QString filename)
{
    //Gettign a pointer on the file
    QFile file(filename);

    //Openign the file
    if(file.open(QIODevice::WriteOnly))  //Opening the file in writing mode

    {          auto text = ui->plainTextEdit->toPlainText();

        QTextStream in(&file);


               in << text ;
           }


    file.close();

}
```
# ***TERMINATED***
