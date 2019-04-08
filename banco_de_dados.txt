#!/usr/bin/env bash
#
# sistema_de_usuarios.sh - Sistema para gerenciamento de usuários com interace garafica
#
# E-mail:     ricardo@servidorcasa.com.br
# Autor:      Ricardo vieira de souza
# Manutenção: Ricardo vieira de souza
#
# ------------------------------------------------------------------------ #
#  Este programa faz todas as funções de gerenciamento de usuários, como:
#  inserir, deletar, alterar.
#
#  Exemplos:
#      $ source sistema_de_usuarios.sh
#      $ ListaUsuarios
# ------------------------------------------------------------------------ #
# Histórico:
#
#   v1.0 07/04/2019 Ricardo
#   v1.1 08/04/2019 Ricardo Bug a ser corrigido = ao clicar em inserir usuario insiro o nome e depois vou em cancelar o carreto
#   e sair do programa porem ele envia a msg de que e necessario incluir um email valido
#
#
#
#
#
#
# ------------------------------------------------------------------------ #

# ------------------------------- VARIÁVEIS ----------------------------------------- #
ARQUIVO_BANCO_DE_DADOS="banco_de_dados.txt"
SEP=:
TEMP=temp.$$
VERDE="\033[32;1m"
VERMELHO="\033[31;1m"
email_REGEX="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$" # essa variavel checa se o e-mail digitado e esta correto



# ------------------------------------------------------------------------ #

# ------------------------------- TESTES ----------------------------------------- #
[ ! -e "$ARQUIVO_BANCO_DE_DADOS" ] && echo "ERRO. Arquivo não existe." && exit 1
[ ! -r "$ARQUIVO_BANCO_DE_DADOS" ] && echo "ERRO. Arquivo não tem permissão de leitura." && exit 1
[ ! -w "$ARQUIVO_BANCO_DE_DADOS" ] && echo "ERRO. Arquivo não tem permissão de escrita." && exit 1
[ ! -x "$(which dialog)" ] && sudo apt install dialog 1> /dev/null 2>&1 #saida para /dev/null
# ------------------------------------------------------------------------ #

# ------------------------------- FUNÇÕES ----------------------------------------- #
  ListaUsuarios () {
    egrep -v "^#|^$" "$ARQUIVO_BANCO_DE_DADOS" | sort -h | tr : ' ' > "$TEMP"
    dialog --title "Lista de Usuários" --textbox "$TEMP" 20 40
    rm -f "$TEMP"
  }

  ValidaExistenciaUsuario () {
    grep -i -q "$1$SEP" "$ARQUIVO_BANCO_DE_DADOS"
  }

  # ------------------------------------------------------------------------ #

  # ------------------------------- EXECUÇÃO ----------------------------------------- #
  while :
  do
    acao=$(dialog --title "Gerenciamento de Usuários 2.0" \
                  --stdout \
                  --menu "Escolha uma das opções abaixo:" \
                  0 0 0 \
                  listar "Listar todos os usuários do sistema" \
                  remover "Remover um usuário do sistema" \
                  inserir "Inserir um novo usuário no sistema")
    [ $? -ne 0 ] && exit

    case $acao in
      listar)  ListaUsuarios  ;;
      inserir)
        ultimo_id=$(egrep -v "^#|^$" "$ARQUIVO_BANCO_DE_DADOS" | sort -h | tail -n 1 | cut -d $SEP -f 1)
        proximo_id=$(($ultimo_id+1))

        nome=$(dialog --title "Cadastro de Usuários" --stdout --inputbox "Digite o seu nome" 0 0)
        [ ! "$nome" ] && continue

        ValidaExistenciaUsuario "$nome" && {
          dialog --title "ERRO FATAL!" --msgbox "Usuário já cadastrado no sistema!" 6 40
          exit 1
        }

        email=$(dialog --title "Cadastro de Usuários" --stdout --inputbox "Digite o seu E-mail" 0 0)
        [ $? -ne 0 ] && continue
        if [ -z "${email}" ]; then
          dialog --title "ERRO!" --msgbox "Necessario Incluir um E-mail!" 6 40
            ListaUsuarios

            else
                if [[ "${email}" =~ ${email_REGEX} ]]; then
                  echo "$proximo_id$SEP$nome$SEP$email" >> "$ARQUIVO_BANCO_DE_DADOS"
                dialog --title "SUCESSO!" --msgbox "Usuário cadastrado com sucesso!" 6 40

              else
                dialog --title "ERRO! " --msgbox "O endereço '${email}' não é válido!" 6 40
                ListaUsuarios
          #exit 2

           fi
          fi

        ListaUsuarios
      ;;
      remover)
        usuarios=$(egrep "^#|^$" -v "$ARQUIVO_BANCO_DE_DADOS" | sort -h | cut -d $SEP -f 1,2 | sed 's/:/ "/;s/$/"/')
        id_usuario=$(eval dialog --stdout --menu \"Escolha um usuário:\" 0 0 0 $usuarios)
        [ $? -ne 0 ] && continue

        grep -i -v "^$id_usuario$SEP" "$ARQUIVO_BANCO_DE_DADOS" > "$TEMP"
        mv "$TEMP" "$ARQUIVO_BANCO_DE_DADOS"

        dialog --msgbox "Usuário removido com sucesso!"
        ListaUsuarios
      ;;
    esac
  done
  # ------------------------------------------------------------------------ #
