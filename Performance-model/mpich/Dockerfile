FROM debian:buster
ADD build/Make.Linux /workdir/
ADD # Copy C program to check MPI rank
ADD build/mpirank.c /workdir
LABEL Author cerl@cray.com --  Cray EMEA Research Lab
LABEL Version v0.1.0
ENV BINDIR=/workdir/bin
ENV MYRANK=$(${BINDIR}/mpirank | cut -d':' -f 1)
ENV NPROCS=$(${BINDIR}/mpirank | cut -d':' -f 2)
ENV NPROCS=${NP}
ENV RUNCMD="mpiexec -np ${NPROCS}"
RUN echo "trace()" >> /entrypoint.sh
RUN echo "{" >> /entrypoint.sh
RUN echo "if test ${MYRANK} -eq 0; then" >> /entrypoint.sh
RUN echo "echo $@" >> /entrypoint.sh
RUN echo "fi" >> /entrypoint.sh
RUN echo "}" >> /entrypoint.sh
RUN echo "timestamp=${timestamp:-$(date '+%Y%m%d%H%M')}" >> /entrypoint.sh
RUN echo "filelabel=${filelabel:-"ranks${NPROCS}"}" >> /entrypoint.sh
RUN echo "maindir=${PWD}" >> /entrypoint.sh
RUN echo "rundir=${rundir:-${maindir}}" >> /entrypoint.sh
RUN echo "trace "NPROCS:" ${NPROCS}" >> /entrypoint.sh
RUN echo "trace "Label file:" ${filelabel}" >> /entrypoint.sh
RUN echo "trace "Rundir:" ${rundir}" >> /entrypoint.sh
RUN echo "run_hpcc()" >> /entrypoint.sh
RUN echo "{" >> /entrypoint.sh
RUN echo "trace "Run:" ${RUNCMD} ${BINDIR}/hpcc" >> /entrypoint.sh
RUN echo "if test ${MYRANK} -eq 0; then" >> /entrypoint.sh
RUN echo "# Delete previous HPCC output" >> /entrypoint.sh
RUN echo "rm -f hpccoutf.txt" >> /entrypoint.sh
RUN echo "test -e $rundir/hpccinf.txt || cp ${maindir}/hpccinf.txt ${rundir}/hpccinf.txt" >> /entrypoint.sh
RUN echo "fi" >> /entrypoint.sh
RUN echo "${RUNCMD} ${BINDIR}/hpcc" >> /entrypoint.sh
RUN echo "if test ${MYRANK} -eq 0; then" >> /entrypoint.sh
RUN echo "# Rename HPCC output" >> /entrypoint.sh
RUN echo "if test -f hpccoutf.txt; then" >> /entrypoint.sh
RUN echo "mv hpccoutf.txt ${maindir}/results/${filelabel}_${timestamp}_hpccoutf.txt" >> /entrypoint.sh
RUN echo "else" >> /entrypoint.sh
RUN echo "trace "Exit with error!"" >> /entrypoint.sh
RUN echo "exit 1" >> /entrypoint.sh
RUN echo "fi" >> /entrypoint.sh
RUN echo "fi" >> /entrypoint.sh
RUN echo "}" >> /entrypoint.sh
RUN echo "run_beffio()" >> /entrypoint.sh
RUN echo "{" >> /entrypoint.sh
RUN echo "TOTMEM=`free -m | grep Mem | awk '{print $2}'`" >> /entrypoint.sh
RUN echo "COREMEM=$(( TOTMEM / $(getconf _NPROCESSORS_ONLN) ))" >> /entrypoint.sh
RUN echo "BEFFFLAGS=${BEFFFLAGS:-"${maindir}/beffio_flags.txt"}" >> /entrypoint.sh
RUN echo "BEFFIOCMD="-MB ${COREMEM} -MT ${TOTMEM} $(cat ${BEFFFLAGS}) -N ${NPROCS} -f ${filelabel}_${timestamp}_b_eff_io"" >> /entrypoint.sh
RUN echo "trace "Run:" ${RUNCMD} ${BINDIR}/b_eff_io ${BEFFIOCMD}" >> /entrypoint.sh
RUN echo "${RUNCMD} ${BINDIR}/b_eff_io ${BEFFIOCMD}" >> /entrypoint.sh
RUN echo "if test ${MYRANK} -eq 0; then" >> /entrypoint.sh
RUN echo "if test -f ${filelabel}_${timestamp}_b_eff_io.prot -a -f ${filelabel}_${timestamp}_b_eff_io.sum; then" >> /entrypoint.sh
RUN echo "mv ${filelabel}_${timestamp}_b_eff_io.prot ${filelabel}_${timestamp}_b_eff_io.sum ${maindir}/results" >> /entrypoint.sh
RUN echo "else" >> /entrypoint.sh
RUN echo "trace "Exit with error!"" >> /entrypoint.sh
RUN echo "exit 1" >> /entrypoint.sh
RUN echo "fi" >> /entrypoint.sh
RUN echo "fi" >> /entrypoint.sh
RUN echo "}" >> /entrypoint.sh
RUN echo "cd ${rundir}" >> /entrypoint.sh
RUN echo "case "${APP}" in" >> /entrypoint.sh
RUN echo "hpcc)" >> /entrypoint.sh
RUN echo "run_hpcc" >> /entrypoint.sh
RUN echo ";;" >> /entrypoint.sh
RUN echo "beffio)" >> /entrypoint.sh
RUN echo "run_beffio" >> /entrypoint.sh
RUN echo ";;" >> /entrypoint.sh
RUN echo "*)" >> /entrypoint.sh
RUN echo "# Run all apps" >> /entrypoint.sh
RUN echo "run_hpcc" >> /entrypoint.sh
RUN echo "run_beffio" >> /entrypoint.sh
RUN echo ";;" >> /entrypoint.sh
RUN echo "esac" >> /entrypoint.sh
RUN echo "cd ${maindir}" >> /entrypoint.sh
RUN chmod u+x /entrypoint.sh
RUN export BINDIR=/workdir/bin
RUN mkdir -p ${BINDIR}
RUN apt-get update && apt-get -y upgrade --no-install-recommends
RUN apt-get -y install --no-install-recommends \
build-essential wget less gfortran procps \
libopenblas-dev
RUN apt-get clean && rm -rf /var/lib/apt/lists/* # do not forget to clean!
WORKDIr /workdir
RUN export MPICH_VERSION=3.3.1
RUN wget -q http://www.mpich.org/static/downloads/${MPICH_VERSION}/mpich-${MPICH_VERSION}.tar.gz
RUN tar xf mpich-${MPICH_VERSION}.tar.gz && rm mpich-${MPICH_VERSION}.tar.gz
WORKDIR /workdir/mpich-${MPICH_VERSION}
RUN ./configure --prefix=/usr/local --disable-static --disable-rpath --disable-wrapper-rpath \
--mandir=/usr/share/man --enable-fast=all,O3
RUN make -j$(getconf _NPROCESSORS_ONLN) install
RUN ldconfig
WORKDIR /workdir
RUN rm -rf mpich-${MPICH_VERSION}
RUN export HPCC_VERSION=1.5.0
RUN wget http://icl.cs.utk.edu/projectsfiles/hpcc/download/hpcc-${HPCC_VERSION}.tar.gz
RUN tar xf hpcc-${HPCC_VERSION}.tar.gz && rm hpcc-${HPCC_VERSION}.tar.gz
WORKDIR hpcc-${HPCC_VERSION}
RUN mv ../Make.Linux hpl/
RUN make arch=Linux
RUN cp /workdir/hpcc-${HPCC_VERSION}/hpcc ${BINDIR}
WORKDIR /workdir
RUN rm -rf hpcc-${HPCC_VERSION}
RUN export BEFFIO_VERSION=2.1
RUN wget --no-check-certificat https://fs.hlrs.de/projects/par/mpi//b_eff_io/b_eff_io_v${BEFFIO_VERSION}.tar.gz
RUN tar xf b_eff_io_v${BEFFIO_VERSION}.tar.gz && rm b_eff_io_v${BEFFIO_VERSION}.tar.gz
WORKDir /workdir/b_eff_io
RUN mpicc -o ${BINDIR}/b_eff_io b_eff_io.c -lm
RUN cp -r eps ${BINDIR}/b_eff_io_eps
WORKDIR /workdir
RUN rm -rf b_eff_io
RUN mpicc /workdir/mpirank.c -o ${BINDIR}/mpirank
CMD /bin/bash /entrypoint.sh
