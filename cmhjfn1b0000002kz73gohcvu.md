---
title: "GhostLLM: How to setup Monaco Editor for QWebEngine"
datePublished: Mon Nov 03 2025 17:47:16 GMT+0000 (Coordinated Universal Time)
cuid: cmhjfn1b0000002kz73gohcvu
slug: ghostllm-how-to-setup-monaco-editor-for-qwebengine
tags: cpp, javascript, opensource, editors, git, qt, monacoeditor

---

Hello,

This blog post will talk about integrating [Monaco Editor](https://microsoft.github.io/monaco-editor/) into a Qt + C++ app using [QWebEngine](https://doc.qt.io/qt-6/qtwebengine-index.html).

# Background

GitRaven is being built to serve as a near-identical replacement to VSCode’s source control management. It aims to offer a similar, yet *opinionated*, user experience.

I personally like side-by-side diff when dealing with file changes. It feels natural to me and I guess, I am *pampered* by VSCode for a couple of years.

In my first GitRaven post, you might have seen a diff viewer in the screenshots. It was built using a custom side-by-side implementation using [KTextEditor](https://invent.kde.org/frameworks/ktexteditor). I gave up on that approach as it was a difficult problem to solve with my current level.

> My future goal (distant) is to replace Monaco based diff viewer with a C++ native side-by-side diff viewer as seen in [Kompare](https://apps.kde.org/kompare/) or Kate’s diff viewers.

# AI Usage

This project uses AI to help me overcome challenges faced in the project. Some code written here has been derived from code snippets shared by LLMs.

# Theory

The idea is to instantiate [QWebEngineView](https://doc.qt.io/qt-6/qwebengineview.html) into the app which loads `index.html` file which contains essential code to initialize Monaco editor as well as setup communication with C++ side as needed.

1. Create a new class `RavenMonaco` which extends from `QWebEngineView` and set it up in main window class.
    
2. Create a new class which extends [QWebEnginePage](https://doc.qt.io/qt-6/qwebenginepage.html) to send commands to Monaco editor. This is primarily used to communicate with Monaco, to init the widget or update diff text when a different file is chosen. We also use it to log messages to `stdout` printed on JS’s `console` for debugging purposes.
    
3. Create the “bridge” class which maintains communication between JS ←—→ C++ side. It is currently used to save text from Monaco’s modified buffer to C++.
    
4. A custom class is needed which handles rendering of Monaco editor on-demand like when a user clicks on diff item from LHS tree. This class is also responsible to save text buffer contents sent in (3) to disk.
    
5. Lastly, we need a class that can serve Monaco code via a HTTP server for enabling [Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers).
    
    > HTTP server is needed for improving performance of Monaco editor. You can find more info in [FAQ section](https://github.com/microsoft/monaco-editor#faq) of the project.
    

#   
Implementation

Let’s start with custom web engine view class.

**ravenmonaco.h**

```cpp
#ifndef RAVENMONACO_H
#define RAVENMONACO_H

#include "ravenmonacopage.h"
#include "ravenmonacohttpserver.h"
#include "ravenmonacobridge.h"

#include <QWebEngineView>
#include <QWidget>
#include <qevent.h>

#include <QJsonObject>

class RavenMonaco : public QWebEngineView
{
    Q_OBJECT
public:
    explicit RavenMonaco(QWidget *parent = nullptr);

    RavenMonacoPage *page() const;

protected:
    void resizeEvent(QResizeEvent *event) override
    {
        // Ensure that the web view resizes dynamically when the parent widget is resized
        resize(event->size()); // Resize webView to match the parent widget
        QWidget::resizeEvent(event);
    }
    void setDefaultUrl();

public slots:
    // Nice to have :)
    void setTheme(Qt::ColorScheme colorScheme);
private:
    RavenMonacoPage *m_page;
    RavenMonacoHTTPServer *m_server;
    RavenMonacoBridge *m_bridge;
    QWebChannel *m_channel;
};

#endif // RAVENMONACO_H
```

This is pretty straightforward. I store references to page, http server, bridge class and channel for future use.

**ravenmonaco.cpp**

```cpp
#include "ravenmonaco.h"

#include <QStyleHints>
#include <QWebChannel>
#include <qmessagebox.h>

namespace fs = std::filesystem;

RavenMonaco::RavenMonaco(QWidget *parent)
    : QWebEngineView{parent}
{
    // Init page
    m_page = new RavenMonacoPage(this);
    setPage(m_page);

    // Light/dark theme switcher
    QStyleHints *hint = QGuiApplication::styleHints();

    // Init monaco when the page load is finished.
    connect(page(), &QWebEnginePage::loadFinished, this, [this](bool ok) {
        if (!ok) {
            qCritical() << "Failed to load Monaco editor, check Monaco HTTP server.";
            QMessageBox errorMsg(QMessageBox::Critical, "GitRaven" , "Failed to load Diff Viewer components.", QMessageBox::Ok);
            errorMsg.exec();
            std::exit(-1);
        }

        // Initialize Monaco internally (doesn't show up in UI yet)
        page()->runJavaScript("init()", 0, [this](const QVariant &) {
            // Update theme
            setTheme(QGuiApplication::styleHints()->colorScheme());
        });
    });

    // Init HTTP server for monaco-editor
    m_server = new RavenMonacoHTTPServer(this);
    m_server->init();

    setDefaultUrl();

    // Init bridge
    m_bridge = new RavenMonacoBridge(this, (RavenEditor*)parent);
    m_channel = new QWebChannel(this);
    m_page->setWebChannel(m_channel);
    // Inform JS side of a JS object available in 
    // `window` that can communicate with C++ world.
    m_channel->registerObject("cppBridge", m_bridge);

    // Light/dark theme switcher
    connect(hint, &QStyleHints::colorSchemeChanged, this, &RavenMonaco::setTheme);
}

RavenMonacoPage *RavenMonaco::page() const
{
    return m_page;
}

void RavenMonaco::setDefaultUrl()
{
    setUrl(QUrl("http://localhost:9191/index.html"));
    load(url());
}

void RavenMonaco::setTheme(Qt::ColorScheme colorScheme)
{
    QJsonObject obj;
    obj["theme"] = colorScheme == Qt::ColorScheme::Light ? "light" : "dark";
    QJsonDocument jd(obj);
    m_page->runJavaScript(QString("setTheme({opt})").replace("{opt}", jd.toJson()));
}
```

This class does a couple of things:

1. Calls `init()` in the JS side to initialize Monaco editor. I want to make the app “ready for rendering diffs” ASAP and so I came up with this logic.
    
2. Then, we setup color scheme detection based on OS’s preference (light/dark) and listen to this event. I personally like this feature.
    

> It’s a (great) gesture from app developers and I appreciate every time an app does it. Most Linux apps follow this by default (at least ones I use) and naturally, I wanted to participate :)

3. We initialize bridge class as well as [QWebChannel](https://doc.qt.io/qt-6/qwebchannel.html) which is the tech magic that enables us to communicate with/from C++ & JS side.
    

**ravenmonacopage.h**

```cpp
#ifndef RAVENMONACOPAGE_H
#define RAVENMONACOPAGE_H

#include <QWebEnginePage>

#include "gitmanager.h"

class RavenMonacoPage : public QWebEnginePage
{
public:
    explicit RavenMonacoPage(QObject *parent = nullptr);
    // This is needed to disable editing Monaco's modified buffer when dealing with
    // staged files.
    void setReadonly(bool readonly);
    // Used to update old/new text content inside Monaco on each file item click.
    void updateText(GitManager::GitDiffItem diffItem);
private:
    void javaScriptConsoleMessage(JavaScriptConsoleMessageLevel level,
                                          const QString &message, int lineNumber,
                                          const QString &sourceID) override;

};

#endif // RAVENMONACOPAGE_H
```

**ravenmonacopage.cpp**

```cpp
#include "ravenmonacopage.h"

#include <QJsonObject>


RavenMonacoPage::RavenMonacoPage(QObject *parent)
    : QWebEnginePage(parent)
{}

void RavenMonacoPage::setReadonly(bool readonly)
{
    runJavaScript(QString("setReadonly({opt})")
    .replace("{opt}", QVariant(readonly).toString()));
}

void RavenMonacoPage::updateText(GitManager::GitDiffItem diffItem)
{
    // Build JSON payload
    // Note: Is there a better way?
    QJsonObject payloadJ;
    payloadJ["oldText"] = diffItem.oldFileContent;
    payloadJ["oldPath"] = diffItem.oldFilePath;
    payloadJ["newText"] = diffItem.newFileContent;
    payloadJ["newPath"] = diffItem.newFilePath;

    QJsonDocument payloadJD(payloadJ);

    QString payloadJDStr = QString(payloadJD.toJson());

    // Send request
    runJavaScript(QString("update({opt})").replace("{opt}", payloadJDStr));
}

void RavenMonacoPage::javaScriptConsoleMessage(
    JavaScriptConsoleMessageLevel level,
    const QString &message,
    int lineNumber, const
    QString &sourceID
)
{
    qDebug() << "RavenMonacoPage::javaScriptConsoleMessage";
    qDebug() << level << message << lineNumber << sourceID;
}
```

**ravenmonacobridge.h**

```cpp
#ifndef RAVENMONACOBRIDGE_H
#define RAVENMONACOBRIDGE_H

#include <QObject>
#include <QDebug>

// Forward declarations
class RavenEditor;

class RavenMonacoBridge : public QObject
{
    Q_OBJECT
public:
    explicit RavenMonacoBridge(QObject *parent = nullptr, RavenEditor *editor = nullptr);
    Q_INVOKABLE void saveModifiedChanges(QString modified);

private:
    RavenEditor *m_ravenEditor;
};

#endif // RAVENMONACOBRIDGE_H
```

**ravenmonacobridge.cpp**

```cpp
#include "ravenmonacobridge.h"

#include "raveneditor.h"

RavenMonacoBridge::RavenMonacoBridge(QObject *parent, RavenEditor *editor)
    : QObject{parent}
{
    m_ravenEditor = editor;
}

/**
 * @brief This function is called by Monaco when user has modified file contents.
 * @param modifiedText - The modified text contents from Monaco side.
 */
void RavenMonacoBridge::saveModifiedChanges(QString modifiedText)
{
    qDebug() << "RavenMonacoBridge::saveModifiedChanges called";

    emit m_ravenEditor->signalSaveModifiedChanges(modifiedText);
}
```

This class exists to trigger `saveModifiedChanges` from JS side and forward it to `RavenEditor` class which actually holds the business logic for processing the request.

Also, here’s the code that allows us to render either a placeholder or Monaco editor as required.

**ravenrhsview.h**

```cpp
#ifndef RAVENRHSVIEW_H
#define RAVENRHSVIEW_H

#include "gitmanager.h"
#include "mainwindow.h"

#include <QVBoxLayout>
#include <QWidget>

class RavenRHSView : public QWidget
{
    Q_OBJECT
public:
    explicit RavenRHSView(QWidget *parent);
    ~RavenRHSView() override;

    void initLandingInfo();

public slots:
    void updateUI(std::optional<GitManager::GitDiffItem> item);

private:
    bool m_showLandingInfo = true;

    MainWindow *m_mainWindow;
    RavenTree   *m_ravenTree;
    RavenEditor *m_ravenEditor;

    QWidget *m_landingInfoWidget;
};

#endif // RAVENRHSVIEW_H
```

**ravenrhsview.cpp**

```cpp
#include "ravenrhsview.h"

#include "raveneditor.h"

#include <QLabel>

RavenRHSView::RavenRHSView(QWidget *parent)
    : QWidget{parent},
    m_mainWindow(static_cast<MainWindow*>(topLevelWidget()->window())),
    m_ravenEditor(new RavenEditor(this)),  // Editor widget
    m_landingInfoWidget(new QWidget(this)) // placeholder widget
{
    // Widget config
    setLayout(new QVBoxLayout(this));
    layout()->addWidget(m_landingInfoWidget);

    updateUI(std::nullopt);

    m_ravenTree = m_mainWindow->getRavenLHSView()->getRavenTree();
    connect(m_ravenTree, &RavenTree::renderDiffItem, this, &RavenRHSView::updateUI);
}

RavenRHSView::~RavenRHSView()
{
    // cleanup
    disconnect(m_ravenTree, &RavenTree::renderDiffItem, this, &RavenRHSView::updateUI);
}

void RavenRHSView::updateUI(std::optional<GitManager::GitDiffItem> item)
{
    qDebug() << "RavenRHSView::updateUI called";

    m_showLandingInfo = !item.has_value();

    // Determine whether we show Diff widget or the placeholder
    if (!m_showLandingInfo) {
        m_landingInfoWidget->hide();
        layout()->addWidget(m_ravenEditor);
        m_ravenEditor->openDiffItem(std::move(item.value()));
    } else {
        initLandingInfo();
    }
}

void RavenRHSView::initLandingInfo()
{
    auto widget = m_landingInfoWidget;

    auto layout = new QGridLayout(widget);
    layout->setAlignment(Qt::AlignCenter);
    widget->setLayout(layout);
    widget->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Expanding);

    auto *label = new QLabel(widget);
    label->setText("GitRaven");
    auto icon = QIcon::fromTheme("git");
    auto iconLabel = new QLabel(widget);
    iconLabel->setPixmap(icon.pixmap(64, 64));

    layout->addWidget(iconLabel);
    layout->addWidget(label);
}
```

**ravenmonacohttpserver.h**

```cpp
#ifndef RAVENMONACOHTTPSERVER_H
#define RAVENMONACOHTTPSERVER_H

#include <QObject>
#include <QHttpServer>
#include <QTcpServer>

class RavenMonacoHTTPServer : QObject
{
    Q_OBJECT
public:
    RavenMonacoHTTPServer(QObject *parent);
    ~RavenMonacoHTTPServer();

    int init();

private:
    QUrl *m_url;
    int PORT = 9191;

    QHttpServer *m_server = new QHttpServer(this);
    QTcpServer *m_tcpserver = new QTcpServer(this);
};

#endif // RAVENMONACOHTTPSERVER_H
```

**ravenmonacohttpserver.cpp**

```cpp
#include "ravenmonacohttpserver.h"

#include <filesystem>

#include "ravenutils.h"

using std::filesystem::absolute;
using std::filesystem::path;

RavenMonacoHTTPServer::RavenMonacoHTTPServer(QObject *parent)
    : QObject(parent) {}

RavenMonacoHTTPServer::~RavenMonacoHTTPServer()
{
    m_tcpserver->close();
}

int RavenMonacoHTTPServer::init()
{
    qDebug() << "RavenMonacoHTTPServer::init() called";

    if (!m_tcpserver->listen(QHostAddress::LocalHost, PORT) || !m_server->bind(m_tcpserver)) {
        qDebug() << "RavenMonacoHTTPServer::init() Failed to bind port for HTTP server.";
        return -1;
    }

    // Listen to / path and return the requested file in URL.
    // Note: Can this be a security risk? For ex: "/index.html/../../../../etc/passwd"
    m_server->route("/<arg>", [] (const QUrl &url) {
        QString urlPath = url.path();

        if (urlPath.length() < 5) {
            return QHttpServerResponse("");
        }

        // Locate editor directory
        path editorDirStdPath = path(RavenUtils::getEditorDirPath());
        path absolutePath = absolute(editorDirStdPath / path(urlPath.toStdString()));

        return QHttpServerResponse::fromFile(QString::fromStdString(absolutePath));
    });

    return 0;
}
```

# Results

**Default state (placeholder)**

![GitRaven default state with placeholder on right-hand side](https://cdn.hashnode.com/res/hashnode/image/upload/v1762190732083/4d08f815-9056-4ecc-aad1-5cc343122690.png align="center")

**Render a diff with Monaco (respecting OS color scheme)**

![GitRaven with Monaco rendering a side-by-side diff on right-hand side](https://cdn.hashnode.com/res/hashnode/image/upload/v1762190741044/1e660ee2-8bbb-4dd6-a3f0-0fb2ab403bfe.png align="center")

# Conclusion

Thanks for your time!

I reluctantly introduced this feature into the project because I couldn’t find a viable alternative to it and building something similar is way too advanced for me at the moment.

Anyway, I hope you liked this post. Please give it a Like to show your appreciation. If you feel there’s something I missed or can do better, let me know in the comments.

Bye for now :-)