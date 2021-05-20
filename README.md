# Python-Unity-Socket
```c#
using System.Collections;
using System.Collections.Generic;
using System.Net;
using System.Net.Sockets;
using System.Text;
using UnityEngine;
using System.Threading;

  public class networkCon : MonoBehaviour {
      Thread mThread;
      public string connectionIP = "127.0.0.1";
      public int connectionPort = 25001;
      IPAddress localAdd;
      TcpListener listener;
      TcpClient client;
      bool running;
      
      // Position is the data being received in this example
      Vector3 position = Vector3.zero;    

      private void Update()
      {
          // Set this object's position in the scene according to the position received
          transform.position = position;
      }

      private void Start()
      {
          // Receive on a separate thread so Unity doesn't freeze waiting for data
          ThreadStart ts = new ThreadStart(GetInfo);
          mThread = new Thread(ts);
          mThread.Start();
      }

      public static string GetLocalIPAddress()
      {
          var host = Dns.GetHostEntry(Dns.GetHostName());
          foreach (var ip in host.AddressList)
          {
              if (ip.AddressFamily == AddressFamily.InterNetwork)
              {
                  return ip.ToString();
              }
          }
          throw new System.Exception("No network adapters with an IPv4 address in the system!");
      }
      
      void GetInfo()
      {
          localAdd = IPAddress.Parse(connectionIP);
          listener = new TcpListener(IPAddress.Any, connectionPort);
          listener.Start();

          client = listener.AcceptTcpClient();
    

          running = true;
          while (running)
          {
              Connection();
          }
          listener.Stop();
      }

      void Connection()
      {
          NetworkStream nwStream = client.GetStream();
          byte[] buffer = new byte[client.ReceiveBufferSize];

          int bytesRead = nwStream.Read(buffer, 0, client.ReceiveBufferSize);
          // Passing data as strings, not ideal but easy to use
          string dataReceived = Encoding.UTF8.GetString(buffer, 0, bytesRead);

          if (dataReceived != null)
          {
              if (dataReceived == "stop")
              {
              // Can send a string "stop" to kill the connection
                  running = false;
              }
              else
              {
                  // Convert the received string of data to the format we are using
                  position = 10f * StringToVector3(dataReceived);
                  print("moved");
                  nwStream.Write(buffer, 0, bytesRead);
              }
          }
      }

      // Use-case specific function, need to re-write this to interpret whatever data is being sent
      public static Vector3 StringToVector3(string sVector)
      {
          // Remove the parentheses
          if (sVector.StartsWith("(") && sVector.EndsWith(")"))
          {
              sVector = sVector.Substring(1, sVector.Length - 2);
          }

          // Split the elements into an array
          string[] sArray = sVector.Split(',');

          // Store as a Vector3
          Vector3 result = new Vector3(
              float.Parse(sArray[0]),
              float.Parse(sArray[1]),
              float.Parse(sArray[2]));

          return result;
      }
  }

```
