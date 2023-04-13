# cs_tcp_trans01


using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;
using System.Net;
using System.Net.Sockets;
using System.Threading;

namespace WinChat
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }
    }

}

class Client
{
    static void client_main(string[] args)
    {
        string ipAddress = "192.168.1.100"; // 相手のIPアドレスを指定
        int port = 1234; // 相手のポート番号を指定
        string message = "Hello, world!"; // 送信するメッセージを指定

        try
        {
            // TCPクライアントを作成し、指定されたIPアドレスとポート番号に接続する
            TcpClient client = new TcpClient(ipAddress, port);

            // メッセージをバイト配列に変換し、TCPストリームに書き込む
            byte[] data = Encoding.ASCII.GetBytes(message);
            NetworkStream stream = client.GetStream();
            stream.Write(data, 0, data.Length);

            // TCPストリームからデータを読み取る
            data = new byte[256];
            int bytes = stream.Read(data, 0, data.Length);
            string responseData = Encoding.ASCII.GetString(data, 0, bytes);
            Console.WriteLine("Received: {0}", responseData);

            // TCPクライアントをクローズする
            client.Close();
        }
        catch (Exception e)
        {
            Console.WriteLine("Exception: {0}", e);
        }

        Console.ReadKey();
    }
}

class Server
{
    static void ServerMain(string[] args)
    {
        int port = 1234; // 待ち受けるポート番号を指定

        try
        {
            // TCPリスナーを作成し、指定されたポート番号で接続を待ち受ける
            TcpListener listener = new TcpListener(IPAddress.Any, port);
            listener.Start();
            Console.WriteLine("Waiting for a connection...");

            // クライアントからの接続を待ち受ける
            TcpClient client = listener.AcceptTcpClient();
            Console.WriteLine("Connected!");

            // クライアントからのデータを受信する
            NetworkStream stream = client.GetStream();
            byte[] data = new byte[256];
            int bytes = stream.Read(data, 0, data.Length);
            string message = Encoding.ASCII.GetString(data, 0, bytes);
            Console.WriteLine("Received: {0}", message);

            // クライアントに応答を送信する
            byte[] responseData = Encoding.ASCII.GetBytes("Message received.");
            stream.Write(responseData, 0, responseData.Length);

            // TCPリスナーとクライアントをクローズする
            client.Close();
            listener.Stop();
        }
        catch (Exception e)
        {
            Console.WriteLine("Exception: {0}", e);
        }

        Console.ReadKey();
    }
}

class Program
{
    static void Main(string[] args)
    {
        int port = 1234; // 待ち受けるポート番号を指定

        try
        {
            // TCPリスナーを作成し、指定されたポート番号で接続を待ち受ける
            TcpListener listener = new TcpListener(IPAddress.Any, port);
            listener.Start();
            Console.WriteLine("Waiting for a connection...");

            // クライアントからの接続を待ち受ける
            TcpClient client = listener.AcceptTcpClient();
            Console.WriteLine("Connected!");

            // 送受信処理を行うスレッドを作成し、開始する
            Thread thread = new Thread(() => DoWork(client));
            thread.Start();

            // メインスレッドを終了させないために、キー入力を待つ
            Console.ReadKey();

            // 送受信処理を行っているスレッドを終了させる
            thread.Abort();
        }
        catch (Exception e)
        {
            Console.WriteLine("Exception: {0}", e);
        }
    }

    // 送受信処理を行うメソッド
    static void DoWork(TcpClient client)
    {
        try
        {
            NetworkStream stream = client.GetStream();

            while (true)
            {
                // クライアントからのデータを受信する
                byte[] data = new byte[256];
                int bytes = stream.Read(data, 0, data.Length);
                string message = Encoding.ASCII.GetString(data, 0, bytes);
                Console.WriteLine("Received: {0}", message);

                // クライアントに応答を送信する
                byte[] responseData = Encoding.ASCII.GetBytes("Message received.");
                stream.Write(responseData, 0, responseData.Length);
            }
        }
        catch (ThreadAbortException)
        {
            // スレッドが終了する前にキャッチする必要がある
            Console.WriteLine("Thread aborted.");
        }
        catch (Exception e)
        {
            Console.WriteLine("Exception: {0}", e);
        }
        finally
        {
            // TCPクライアントをクローズする
            client.Close();
        }
    }
}
