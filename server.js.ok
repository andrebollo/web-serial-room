// server.js
const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = new Server(server);

app.use(express.static("public"));

const rooms = {};

io.on("connection", (socket) => {
  console.log("Cliente conectado:", socket.id);

  socket.on("joinRoom", ({ room, name }) => {
    socket.join(room);
    socket.room = room;
    socket.name = name;

    if (!rooms[room]) {
      rooms[room] = { users: {}, host: null, serialHost: null, text: "" };
    }
    rooms[room].users[socket.id] = name;

    socket.emit("syncText", rooms[room].text);

    io.to(room).emit("updateUsers", {
      users: rooms[room].users,
      host: rooms[room].host,
      serialHost: rooms[room].serialHost,
    });
    io.to(room).emit("typingLog", `${name} entrou na sala.`);
  });

  socket.on("setHost", ({ room }) => {
    if (!rooms[room]) return;
    rooms[room].host = socket.id;
    io.to(room).emit("updateUsers", {
      users: rooms[room].users,
      host: rooms[room].host,
      serialHost: rooms[room].serialHost,
    });
    io.to(room).emit("typingLog", `${rooms[room].users[socket.id]} agora é host serial.`);
  });

  socket.on("serialConnected", ({ room, name }) => {
    if (!rooms[room]) return;
    rooms[room].serialHost = name;
    io.to(room).emit("serialStatus", `Conectado por ${name}`);
    io.to(room).emit("updateUsers", {
      users: rooms[room].users,
      host: rooms[room].host,
      serialHost: rooms[room].serialHost,
    });
  });

  socket.on("keyPress", ({ room, data }) => {
    if (!rooms[room]) return;
    const userName = rooms[room].users[socket.id] || "unknown";

    // envia para o host se existir
    if (rooms[room].host) {
      io.to(rooms[room].host).emit("hostWrite", { data, from: userName });
    }

    io.to(room).emit("typingLog", `${userName} >> ${JSON.stringify(data)}`);
  });

  socket.on("serialIn", ({ room, data }) => {
    if (!rooms[room]) return;

    rooms[room].text += data;
    io.to(room).emit("appendText", data);
    io.to(room).emit("serialLog", `<< ${data}`);
  });

  socket.on("disconnect", () => {
    const room = socket.room;
    if (!room || !rooms[room]) return;

    const name = rooms[room].users[socket.id];
    delete rooms[room].users[socket.id];

    if (rooms[room].host === socket.id) rooms[room].host = null;
    if (rooms[room].serialHost === name) rooms[room].serialHost = null;

    io.to(room).emit("typingLog", `${name} saiu.`);
    io.to(room).emit("updateUsers", {
      users: rooms[room].users,
      host: rooms[room].host,
      serialHost: rooms[room].serialHost,
    });

    if (Object.keys(rooms[room].users).length === 0) {
      delete rooms[room];
    }
  });
});

server.listen(3000, () => console.log("Servidor rodando em http://localhost:3000"));
