# CameraUDPConnector
import socket
import cv2
import numpy as np

BUFF_SIZE = 65535
CHUNK_SIZE = 16384
WIDTH = 640
HEIGHT = 480
port = 5555

# Set the end marker
delimiter = b'FRAME_END_MARKER'

try:
    # Create the UDP socket
    UDP_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    UDP_socket.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, BUFF_SIZE)

    host_name = socket.gethostname()
    host_ip = "0.0.0.0"
   
    # Bind the socket to listen for incoming video stream
    UDP_socket.bind((host_ip, port))
    print(f"UDP socket bound to {host_ip}:{port}")

    # Create a window for displaying video
    cv2.namedWindow("Received Video Feed", cv2.WINDOW_NORMAL)

    while True:
        # Buffer to store received frame chunks
        frame_data = bytearray()

        while True:
            # Receive chunks of the video stream
            chunk, addr = UDP_socket.recvfrom(CHUNK_SIZE)
            if chunk == delimiter:
                print("End of frame received")
                break  # End of current frame, exit the loop
           
            # Append chunk to the frame data buffer
            frame_data.extend(chunk)

        # Decode the frame data into an image
        frame = cv2.imdecode(np.frombuffer(frame_data, dtype=np.uint8), cv2.IMREAD_COLOR)
       
        # If frame is valid, display it
        if frame is not None:
            frame = cv2.resize(frame, (WIDTH, HEIGHT))
            cv2.imshow("Received Video Feed", frame)

        # Exit condition: press 'q' to quit
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

finally:
    print("Closing the socket and cleaning up.")
    UDP_socket.close()
    cv2.destroyAllWindows()
