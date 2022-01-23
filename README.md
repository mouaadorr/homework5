# *ToDo Application using containers*
----
&nbsp;
# Introduction
The Qt library provides a set of general purpose template-based container classes. These classes can be used to store items of a specified type. For example, if you need a resizable array of QStrings, use QVector< QString >.

These container classes are designed to be lighter, safer, and easier to use than the STL containers. If you are unfamiliar with the STL, or prefer to do things the "Qt way", you can use these classes instead of the STL classes.

&nbsp;

> We have already checked three basic containers:
* list : list is a container that supports constant time insertion and removal of elements from anywhere in the container.
   
* Tree : to manage a collection in the form of a tree.
  
* Table : to manage widgets in 2D lattice array.

&nbsp;

And we have also seen the difference between **Item Based** and **Model Based**, also each one features.

&nbsp;

# Overview

*The goal of the homework is to create an application to manage your tasks. It should have all the features of main application such as menues, actions and toolbar. The application must store an archive of all the pending and finished tasks.*

&nbsp;

# Use Cases
* A user should be able to close the application of course.
* A Todo application cannot be useful, unless it offers the possibility of creating new tasks.
   * The essential components of a task will be defined later
* The View of the main widget should be split in three areas:

  * The first (en persistent) area shows the list of today tasks.
  * The second one is reserved for pending task (tasks for the future).
  * Finally, the third one shows the set of finished tasks.
* Each category must be shown with a custom icon.

* The user could either hide/show the pending and finished views.

* Finally, the tasks entered to your application must remains in the app in future use.

&nbsp;

# Defining a Task

A Task is defined by the following attributes:

* A description: stating the text and goal for the task like (Buying the milk).

* A finished boolean indicating if the task is Finished or due.
* A Tag category to show the class of the task which is reduced to the following values:

   * Work
   * Life
   * Other
* Finally, a task should have a DueDate which stores the Date planned for the date.

&nbsp;

> # Let's code now
&nbsp;

# **I will start with the MVC Model version** 
----

&nbsp;

# *Mainwindow Interface*
![](/images/mainwindow.PNG)

&nbsp;
# *Defining Task Interface*
![](/images/dialog.PNG)


&nbsp;

> # newddialog.h
```cpp
#ifndef NEWDDIALOG_H
#define NEWDDIALOG_H
#include <QComboBox>
#include <QDateEdit>
#include <QDialog>
#include<QtDebug>

namespace Ui {
class newddialog;
}

class newddialog : public QDialog
{
    Q_OBJECT

public:
    explicit newddialog(QWidget *parent = nullptr);
    ~newddialog();
    // variable that return true if the ok is clicked or false if the cancel is clicked
    bool logic=false;

    private slots:

   //slot for the pushbutton --> OK
    void on_OK_clicked();
   //slot for the pushbutton --> Cancel
    void on_pushButton_2_clicked();

public:
    Ui::newddialog *ui;
};

#endif // NEWDDIALOG_H
```
> # newddialog.cpp
```cpp
#include "newddialog.h"
#include "ui_newddialog.h"

newddialog::newddialog(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::newddialog)
{
    ui->setupUi(this);

}

newddialog::~newddialog()
{
    delete ui;
}


void newddialog::on_OK_clicked()
{
    logic=true;
    close();
}

void newddialog::on_pushButton_2_clicked()
{
    logic=false;
    close();
}
```
&nbsp;

> # MainWindow.h
```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H
#include <QMainWindow>
#include <QStandardItemModel> //<--- class provides a generic model for storing custom data
#include <QFile>
#include<QStringListModel>
#include <QFileDialog>
#include <QTextStream>
#include <QCloseEvent>
#include <QMessageBox>
#include <QMouseEvent>
#include "newddialog.h"


QT_BEGIN_NAMESPACE
namespace Ui { class MainWindow; }
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT

private slots:

   // slot for the new task
    void on_actionNew_triggered();
   //slot to delete a task
    void on_actionDelete_triggered();
   //slot to show only pending task
    void on_actionPending_triggered();
    //slot to show only finished task
    void on_actionFinished_triggered();
   //slot to show only today task
    void on_actionToday_triggered();
   //slot to exit the application
    void on_actionExit_triggered();
   //slot to show today, pending and finished tasks
    void on_actionShowall_triggered();
  //slot to close the application
    void closeEvent (QCloseEvent *event);

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();
    //standaritems to store our data
    QStandardItemModel *model=nullptr;
    QStandardItemModel *mod2=nullptr;
    QStandardItemModel *mod3=nullptr;
    QStandardItemModel *mod4=nullptr;


     //text files that will contain our tasks
      QFile  today_task{"Today tasks.txt"};
      QFile finished_task{"Finished tasks.txt"};
      QFile pending_task{"Pending tasks.txt"};

      //function that will load files
      void load(QFile *filename=nullptr);

      //function to save finished tasks tasks
      void saveFinishedtasks(QFile *filename=nullptr) const;
      //function to save today tasks tasks
      void saveTodaytasks(QFile *filename=nullptr) const;
      //function to save pending tasks tasks
      void savePendingtasks(QFile *filename=nullptr) const;

      QStringList Today;
      QStringList Finished;
      QStringList Pending;

public:
    Ui::MainWindow *ui;
};
#endif // MAINWINDOW_H
```
&nbsp;
> # MainWindow.cpp
```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include "newddialog.h"
#include "ui_newddialog.h"
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    //create a model
    QStandardItem *model = new QStandardItem;

    mod2 = new QStandardItemModel();
    mod3 = new QStandardItemModel();
    mod4 = new QStandardItemModel();

    QFile today_task{"Today tasks.txt"};
    QFile finished_task{"Finished tasks.txt"};
    QFile pending_task{"Pending tasks.txt"};

    model->setText("");
    mod2->appendRow(model);
    ui->listView->setModel(mod2);
    mod3->appendRow(model);
    ui->listView_2->setModel(mod3);
    mod4->appendRow(model);
    ui->listView_3->setModel(mod4);

    mod2->removeRow(0);
    mod3->removeRow(0);
    mod4->removeRow(0);


      load(&today_task);
      load(&finished_task);
      load(&pending_task);


setAcceptDrops(false);

// add a style to each task of each list
ui->listView->setStyleSheet("QListView{font-size: 10pt; font-family: Lucida Console, Courier New, monospace; }" "QListView::item { background-color: #F0E68C; pandding: 30%;"
                                "border: 1px solid #FFFFFF;}"
                                "QListView:: item::hover{background-color: #F0E68C}");
ui->listView_2->setStyleSheet("QListView{font-size: 10pt;font-family: Lucida Console, Courier New, monospace;}" "QListView::item { background-color: #87CEFA; pandding: 30%;"
                                "border: 1px solid #FFFFFF;}"
                                "QListView:: item::hover{background-color: #87CEFA;}");
ui->listView_3->setStyleSheet("QListView{font-size: 10pt;font-family: Lucida Console, Courier New, monospace; }" "QListView::item { background-color: #7CFC00; pandding: 30%;"
                                "border: 1px solid #FFFFFF;}"
                                "QListView:: item::hover{background-color: #7CFC00}");

ui->listView->setAcceptDrops(true);
ui->listView_2->setAcceptDrops(true);
ui->listView_3->setAcceptDrops(true);

}
MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::on_actionNew_triggered()
{

    newddialog newD;
    //get the current date
    newD.ui->dateEdit->setDate(QDate::currentDate());
    //show the dialog
    newD.exec();
    // create a standaritem
      QStandardItem *it = new QStandardItem();


    // if the ok is clicked
    if(newD.logic){

           // see if the the checkbox is checked
           if(newD.ui->checkBox->isChecked()){
             // add task to stringlist -->finished
              Finished.append(newD.ui->lineEdit->text()+" Due: "+ newD.ui->dateEdit->date().toString("ddd MMMM d yyyy")+" Tag: "+ newD.ui->comboBox->currentText());
              //set the icon
              it->setIcon(QPixmap(":/icons/finished.png"));
              //add the taks to the item
              it->setText(newD.ui->lineEdit->text()+" Due: "+ newD.ui->dateEdit->date().toString("ddd MMMM d yyyy")+" Tag: "+ newD.ui->comboBox->currentText());
              //add the item
              mod4->appendRow(it);
              // set the model
              ui->listView_3->setModel(mod4);
              //finally update the list
              ui->listView_3->update();



           }
           else if(newD. ui->dateEdit->date().toString("ddd MMMM d yyyy")==QDate::currentDate().toString("ddd MMMM d yyyy")){
               Today.append(newD.ui->lineEdit->text()+" Due: "+ newD.ui->dateEdit->date().toString("ddd MMMM d yyyy")+" Tag: "+ newD.ui->comboBox->currentText());
               it->setIcon(QPixmap(":/icons/today.png"));
               it->setText(newD.ui->lineEdit->text()+" Due: "+ newD.ui->dateEdit->date().toString("ddd MMMM d yyyy")+" Tag: "+ newD.ui->comboBox->currentText());
               mod2->appendRow(it);
               ui->listView->setModel(mod2);
               ui->listView->update();

           }
           else{
               Pending.append(newD.ui->lineEdit->text()+" Due: "+ newD.ui->dateEdit->date().toString("ddd MMMM d yyyy")+" Tag: "+ newD.ui->comboBox->currentText());
               it->setIcon(QPixmap(":/icons/pending.png"));
               it->setText(newD.ui->lineEdit->text()+" Due: "+ newD.ui->dateEdit->date().toString("ddd MMMM d yyyy")+" Tag: "+ newD.ui->comboBox->currentText());
               mod3->appendRow(it);
               ui->listView_2->setModel(mod3);
               ui->listView_2->update();
           }
    newD.close();

      }
    }
void MainWindow::on_actionDelete_triggered()
{
    // if there's at least a row in the list
    if( ui->listView->model()->rowCount()!=0){
        QModelIndexList indexes1;
        //get the selected index
        indexes1= ui->listView->selectionModel()->selectedIndexes();
        // loop through the stringlist--> Today and remove the selected item at corresponding index
        for(auto i = indexes1.constBegin(); i != indexes1.constEnd(); ++i){
           Today.removeAt((*i).row());
           mod2->removeRow((*i).row());
            }}

    if (ui->listView_2->model()->rowCount()!=0){

        QModelIndexList indexes2;
        indexes2 = ui->listView_2->selectionModel()->selectedIndexes();

        for(auto i = indexes2.constBegin(); i != indexes2.constEnd(); ++i){
            Pending.removeAt((*i).row());

           mod3->removeRow((*i).row());
        }}

    if(ui->listView_3->model()->rowCount()!=0){
        QModelIndexList indexes3;
        indexes3 = ui->listView_3->selectionModel()->selectedIndexes();

        for(auto i = indexes3.constBegin(); i != indexes3.constEnd(); ++i){
            Finished.removeAt((*i).row());

           mod4->removeRow((*i).row());
        }
    }
}

void MainWindow::on_actionPending_triggered()
{
    ui->listView->hide();
    ui->listView_3->hide();
    ui->listView_2->show();
}


void MainWindow::on_actionFinished_triggered()
{
    ui->listView->hide();
    ui->listView_2->hide();
    ui->listView_3->show();

}


void MainWindow::on_actionToday_triggered()
{
    ui->listView_2->hide();
    ui->listView_3->hide();
    ui->listView->show();

}


void MainWindow::on_actionExit_triggered()
{
    // a message that shown by Qt asking whether you want exit or not
    auto reply = QMessageBox::question(this, "Exit",
                                       "Do you really want to quit?");
    //if the response is yes, then ----> EXIT
    if(reply == QMessageBox::Yes){
        //but before quit the application save all tasks
        saveTodaytasks(&today_task);
            saveFinishedtasks(&finished_task);
            savePendingtasks(&pending_task);
            qApp->exit();
    }
}
void MainWindow::load(QFile *filename){
    //If the file was successfully opened for reading
    if (filename->open(QIODevice::ReadOnly)) {
        //Initiating a stream using the file
        QTextStream in(filename);

        if(filename->fileName()==finished_task.fileName()){
            while (!in.atEnd()) {
                QStandardItem *it = new QStandardItem();

                QString line = in.readLine();
            Finished.append(line);
            it->setIcon(QPixmap(":/icons/finished.png"));
            it->setText(line);
              mod4->appendRow(it);
              ui->listView_3->setModel(mod4);
       }

        }
        else if(filename->fileName()==today_task.fileName()){
            while (!in.atEnd()) {
                QStandardItem *it = new QStandardItem();

                QString line = in.readLine();
             Today.append(line);
             it->setIcon(QPixmap(":/icons/today.png"));
             it->setText(line);
             mod2->appendRow(it);
                 ui->listView->setModel(mod2);
}
  }
        else if(filename->fileName()==pending_task.fileName()){
            while (!in.atEnd()) {
                QStandardItem *it = new QStandardItem();

                QString line = in.readLine();
                Pending.append(line);
                it->setIcon(QPixmap(":/icons/pending.png"));
                it->setText(line);
                mod3->appendRow(it);
                ui->listView_2->setModel(mod3);
  }
        }
    }

}
void MainWindow::saveTodaytasks(QFile *filename) const{

    //Openign the file
    if(filename->open(QIODevice::WriteOnly))  //Opening the file in writing mode
    {
        //Initiating a stream using the file
        QTextStream out(filename);

        for ( int i=0; i < Today.size(); ++i )
                        out << Today.at(i) << "\n";

    }


    filename->close();

}
void MainWindow::savePendingtasks(QFile *filename) const{
    //Openign the file
    if(filename->open(QIODevice::WriteOnly))  //Opening the file in writing mode
    {
        //Initiating a stream using the file
        QTextStream out(filename);

        for ( int i=0; i < Pending.size(); ++i )
                        out << Pending.at(i) << "\n";
    }

    filename->close();
}
void MainWindow::saveFinishedtasks(QFile *filename) const{
    //Openign the file
    if(filename->open(QIODevice::WriteOnly))  //Opening the file in writing mode
    {
        //Initiating a stream using the file
        QTextStream out(filename);

        for ( int i=0; i < Finished.size(); ++i )
                        out << Finished.at(i) << "\n";
    }

    filename->close();

}

void MainWindow::on_actionShowall_triggered()
{
    ui->listView->show();
    ui->listView_2->show();
    ui->listView_3->show();
}

void MainWindow::closeEvent(QCloseEvent *event){
    QMessageBox::StandardButton cls = QMessageBox::question( this, "TODO",
                                                                tr("do you want to quit?\n"),
                                                                QMessageBox::No | QMessageBox::Yes,
                                                                QMessageBox::Yes);
    if (cls != QMessageBox::Yes) {
        event->ignore();
    } else {
        //call the slot of Exit
        on_actionExit_triggered();
        event->accept();
    }
}
```
&nbsp;
# *check the result*


&nbsp;

# **Item Based Model Version**
---

&nbsp;
# *Mainwindow interface* 
![](/images/mainwindow2.PNG)
&nbsp;

#  *Dialog interface*
![](/images/dialog2.PNG)
&nbsp;
> # addTaskDialog.h
```cpp
#ifndef ADDTASKDIALOG_H
#define ADDTASKDIALOG_H

#include <QDialog>

namespace Ui {
class addTaskDialog;
}

class addTaskDialog : public QDialog
{
    Q_OBJECT

public:
    explicit addTaskDialog(QWidget *parent = nullptr);
    QString getText();
    ~addTaskDialog();
    QDate getDate();
    bool statusCheckbox();
    void setdate(int y,int m, int d);
    void setdesc(QString a);
    void settag(QString a);
    void setf(bool a);


private:
    Ui::addTaskDialog *ui;
};

#endif // ADDTASKDIALOG_H
```
&nbsp;

> # addTaskDialog.cpp
```cpp
#include "addtaskdialog.h"
#include "ui_addtaskdialog.h"
#include "QDate"
addTaskDialog::addTaskDialog(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::addTaskDialog)
{
    ui->setupUi(this);
    QDate date =QDate::currentDate();
    ui->dateEdit->setMinimumDate(date);
    ui->dateEdit->setDate(date);
}

addTaskDialog::~addTaskDialog()
{
    delete ui;
}
void addTaskDialog::setdate(int y,int m, int d){
    QDate da;

    da.setDate(y,m,d);
    ui->dateEdit->setDate(da);
}
void addTaskDialog::setdesc(QString a){
    ui->lineEdit->setText(a);
}
void addTaskDialog::settag(QString a){
    ui->comboBox->setCurrentText(a);
}
void addTaskDialog::setf(bool a){
    ui->checkBox->setChecked(a);
}


bool addTaskDialog::statusCheckbox(){
   return ui->checkBox->isChecked();
}
QDate addTaskDialog::getDate(){
    return ui->dateEdit->date();
}
QString addTaskDialog::getText(){

    QString task= ui->lineEdit->text() + " Due : " + ui->dateEdit->text() + "  Tag : " + ui->comboBox->currentText() +"";
    return task;
}
```
&nbsp;
> # todoapp.h
```cpp
#ifndef TODOAPP_H
#define TODOAPP_H

#include <QMainWindow>
#include <QFile>
#include <QCloseEvent>
#include <QDropEvent>
QT_BEGIN_NAMESPACE
namespace Ui { class ToDoApp; }
QT_END_NAMESPACE

class ToDoApp : public QMainWindow
{
    Q_OBJECT

public:
    ToDoApp(QWidget *parent = nullptr);
    ~ToDoApp();
   void makeConnexion();

private slots:

    void list1();
    void list2();
    void list3();

    void on_action_Exit_triggered();
    void on_actionPending_toggled(bool arg1);

    void on_actionFinished_toggled(bool arg1);

    void on_actionTask_triggered();

private:
    Ui::ToDoApp *ui; QFile file;
protected:
    void closeEvent(QCloseEvent* e) override;
    
};
#endif // TODOAPP_H
```
> # todoapp.cpp
```cpp
#include "todoapp.h"
#include "ui_todoapp.h"
#include "addtaskdialog.h"
#include<QDate>
#include<QFile>
#include <QTextStream>
#include<QApplication>
#include <iostream>
#include <QMessageBox>

ToDoApp::ToDoApp(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::ToDoApp)
{
      ui->setupUi(this);
      QFile file("C:/Users/Mouaad/Desktop/store.txt");

      if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
            return;

      while (!file.atEnd()) {
          QString line = file.readLine();
          if(line.at(0)=="1"){
                ui->listWidget->addItem(line.mid(1,line.size()));
          }else if(line.at(0)=="2"){
              ui->listWidget2->addItem(line.mid(1,line.size()));
          }else if(line.at(0)=="3"){
              ui->listWidget3->addItem(line.mid(1,line.size()));
          }
      }


      ui->listWidget->setDragEnabled(true);
      ui->listWidget->setAcceptDrops(true);
      ui->listWidget->setDropIndicatorShown(true);
      ui->listWidget->setDefaultDropAction(Qt::MoveAction);

      // -----------//
      ui->listWidget2->setVisible(false);
      ui->listWidget2->setDragEnabled(true);
      ui->listWidget2->setAcceptDrops(true);
      ui->listWidget2->setDropIndicatorShown(true);
      ui->listWidget2->setDefaultDropAction(Qt::MoveAction);
         // -----------//
      ui->listWidget3->setVisible(false);
      ui->listWidget3->setDragEnabled(false);
      ui->listWidget3->setAcceptDrops(true);
      ui->listWidget3->setDropIndicatorShown(true);
      ui->listWidget3->setDefaultDropAction(Qt::MoveAction);
        //-----------//


}

ToDoApp::~ToDoApp()
{
    delete ui;
}

void ToDoApp::makeConnexion(){
    connect(ui->listWidget, SIGNAL(itemDoubleClicked(QListWidgetItem*)), this,SLOT(list1()));
    connect(ui->listWidget2, SIGNAL(itemDoubleClicked(QListWidgetItem*)), this,SLOT(list2()));
    connect(ui->listWidget3, SIGNAL(itemDoubleClicked(QListWidgetItem*)), this,SLOT(list3()));

}

void ToDoApp::list1(){
    addTaskDialog dialog;
    int i=1;
    QString task;
    QListWidgetItem *item=ui->listWidget->currentItem();
    QStringList list = item->text().split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];

    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }

    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.setdesc(task);
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();


     if(reply == addTaskDialog::Accepted)
     {
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.statusCheckbox()){
             ui->listWidget->addItem(text);
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.statusCheckbox()){
             ui->listWidget2->addItem(text);
         }else if(dialog.statusCheckbox()){
             ui->listWidget3->addItem(text);
         }
         delete item ;
     }
}
void ToDoApp::list2 (){
    addTaskDialog dialog;
    int i=1;
    QString task;
    QListWidgetItem *item=ui->listWidget2->currentItem();
    QStringList list = item->text().split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];
    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.setdesc(task);
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted){
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.statusCheckbox()){
             ui->listWidget->addItem(text);
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.statusCheckbox()){
             ui->listWidget2->addItem(text);
         }else if(dialog.statusCheckbox()){
             ui->listWidget3->addItem(text);
         }
         delete item ;
    }
}
void ToDoApp::list3(){
    addTaskDialog dialog;
    int i=1;
    QString task;
    QListWidgetItem *item=ui->listWidget3->currentItem();
    QStringList list = item->text().split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];
    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.setdesc(task);
    dialog.setf(true);
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted){
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.statusCheckbox()){
             ui->listWidget->addItem(text);
         }else if(dialog.getDate()!=QDate::currentDate()  && !dialog.statusCheckbox()){
             ui->listWidget2->addItem(text);
         }else if(dialog.statusCheckbox()){
             ui->listWidget3->addItem(text);
         }
         delete item ;
     }
}

void ToDoApp::on_actionTask_triggered()
{
    addTaskDialog dialog;
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted)
    {
        QString text= dialog.getText();
        if(dialog.getDate()==QDate::currentDate() && !dialog.statusCheckbox()){
            ui->listWidget->addItem(text);
        }else if(dialog.getDate()!=QDate::currentDate() && !dialog.statusCheckbox()){
            ui->listWidget2->addItem(text);
        }else if(dialog.statusCheckbox()){
            ui->listWidget3->addItem(text);
        }
    }
}


void ToDoApp::on_action_Exit_triggered()
{
       // a message that shown by Qt asking whether you want exit or not
            auto reply = QMessageBox::question(this, "Exit",
                                               "Do you really want to quit?");
            //if the response is yes, then ----> EXIT
            if(reply == QMessageBox::Yes)
                qApp->exit();
}

void ToDoApp::closeEvent(QCloseEvent* event){
    QFile file("C:/Users/Mouaad/Desktop/store.txt");
    if(file.open(QIODevice::ReadWrite | QIODevice::Text)){
        QTextStream out(&file);
        for(int i=0;i<ui->listWidget->count();i++)
        {
            out << "1"<< ui->listWidget->item(i)->text() << Qt::endl;
        }
        for(int i=0;i<ui->listWidget2->count();i++)
        {
            out << "2"<< ui->listWidget2->item(i)->text() << Qt::endl;
        }
        for(int i=0;i<ui->listWidget3->count();i++)
        {
            out << "3"<< ui->listWidget3->item(i)->text() << Qt::endl;
        }
        file.close();
    }
    event->accept();
}

void ToDoApp::on_actionPending_toggled(bool arg1)
{
    if(arg1){
        ui->listWidget2->setVisible(true);
    }else{
        ui->listWidget2->setVisible(false);
    }
}


void ToDoApp::on_actionFinished_toggled(bool arg1)
{
    if(arg1){
        ui->listWidget3->setVisible(true);
    }else{
        ui->listWidget3->setVisible(false);
    }

}
```
&nbsp;

# *check the result*





----
> # Conclusion 
The Qt Framework handles the MVC pattern implicitly, especially when we work with pre-built APIs of the model, view, and delegation classes.Combined with the trend of front-end design in recent years, the final choice is QT + Vue + element UI + SQLite (the database is selected according to the needs)

* QT is responsible for interface and hardware processing
* SQLite for data storage
* Vue + element UI implements the front end








